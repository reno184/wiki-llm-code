# Upload de fichier — backend (Express + busboy, streaming multipart)

> Miroir local de [backend/upload-file-busboy.md](https://github.com/reno184/wiki-llm-code/blob/main/backend/upload-file-busboy.md) — source de vérité : repo GitHub wiki-llm-code.

Stack: Node 20+ / Express (ESM) / busboy.
Usage: recevoir un upload multipart/form-data dans une route Express, streamer le fichier, valider, déléguer le stockage à un driver (Strategy). Fiche back du trio upload fullstack (voir [upload-file.md](upload-file.md) et [upload-strategy.md](upload-strategy.md)).

```bash
npm install busboy
```

## Pourquoi busboy (et pas multer)

- **Streaming** : busboy émet le fichier en chunks au fur et à mesure — pas de bufferisation de tout le fichier en mémoire avant de savoir s'il est valide.
- On peut **interrompre l'upload** dès qu'une limite est dépassée (taille, type) au lieu d'absorber 200 Mo inutiles.
- Multer écrit sur disque par défaut et tamponne en mémoire sinon ; busboy laisse le contrôle total.

## Middleware de validation du Content-Type (avant tout parsing)

Rejette la requête **avant** de lancer busboy si le header n'est pas un multipart conforme. Fail fast : un mauvais Content-Type ne doit jamais arriver jusqu'au stream.

```js
// middlewares/uploadValidator.js
export const uploadValidator = (req, res, next) => {
  const contentType = req.headers['content-type']

  if (!contentType) {
    return next(new Error('Content-Type header missing'))
  }

  // Extraire le boundary du Content-Type multipart
  // Format : "multipart/form-data; boundary=----WebKitFormBoundary..."
  const isMultipart = contentType.includes('multipart/form-data')

  if (!isMultipart) {
    return next(new Error('Expected multipart/form-data'))
  }

  next()
}
```

## Route upload — pattern

Le middleware se branche **dans la chaîne de la route**, avant le handler busboy :

```js
import express from 'express'
import Busboy from 'busboy'
import { uploadValidator } from '../middlewares/uploadValidator.js'
import { getUploadDriver } from '../drivers/index.js' // Strategy : driver choisi via env

const router = express.Router()

const MAX_SIZE = 20 * 1024 * 1024 // 20 Mo — même limite que le front
const ALLOWED_TYPES = ['application/pdf', 'image/jpeg', 'image/png', 'image/webp']

router.post('/api/upload', uploadValidator, (req, res, next) => {
  const busboy = Busboy({
    headers: req.headers,
    limits: { fileSize: MAX_SIZE }, // busboy tronque au-delà : détecté via file.truncated
  })

  const fields = {}
  let uploadPromise = null

  busboy.on('field', (name, value) => { fields[name] = value })

  busboy.on('file', (name, stream, info) => {
    const { filename, mimeType } = info

    if (!ALLOWED_TYPES.includes(mimeType)) {
      stream.resume() // drainer le stream, sinon la requête reste pendante
      uploadPromise = Promise.reject(new Error('Type de fichier non autorisé.'))
      return
    }

    // key déterministe anti-collision (cf. fiche minio : <timestamp>-<baseFilename>)
    const objectName = `${Date.now()}-${filename.replace(/[^a-zA-Z0-9._-]/g, '_')}`

    uploadPromise = getUploadDriver().put(stream, objectName, mimeType, MAX_SIZE)
      .then(({ url, key }) => ({ url, key }))
  })

  busboy.on('error', next)
  busboy.on('finish', async () => {
    try {
      if (!uploadPromise) throw new Error('Aucun fichier reçu.')
      const result = await uploadPromise
      res.status(201).json({ ...result, fields })
    } catch (err) {
      next(err) // pas de fallback : l'erreur remonte au middleware global
    }
  })

  req.pipe(busboy) // ⚠️ c'est ça qui lance le parsing
})

export default router
```

## Driver MinIO compatible stream

Réutilise le singleton client de [minio-storage-express.md](minio-storage-express.md) ; busboy donne un **stream**, `putObject` l'accepte directement (pas besoin de bufferiser) :

```js
// drivers/minio-driver.js
import { minioClient, BUCKET_NAME } from '../config/minio.js'

export const put = (stream, objectName, mimeType, maxFileSize) =>
  new Promise((resolve, reject) => {
    let size = 0
    stream.on('data', (chunk) => {
      size += chunk.length
      if (size > maxFileSize) {
        stream.destroy() // stoppe la lecture : le reste de l'upload n'est pas absorbé
        reject(new Error('Fichier trop volumineux.'))
      }
    })
    stream.on('error', reject)
    minioClient.putObject(BUCKET_NAME, objectName, stream, undefined, {
      'Content-Type': mimeType,
    }).then(() => resolve({ url: `${BUCKET_NAME}/${objectName}`, key: objectName }))
       .catch(reject)
  })
```

## Notes

- **`uploadValidator` protège l'entrée de la chaîne** : header absent ou non-multipart → rejet avant tout parsing. Il ne remplace ni les limites de taille (driver) ni la validation MIME par fichier (handler busboy) — chaque couche garde son rôle.
- Un middleware Express = une fonction `(req, res, next)` placée dans la chaîne de la route (`router.post(path, uploadValidator, handler)`) ou en `app.use` global. `next(err)` saute au middleware d'erreur global ; **toujours `return` après `next(err)`** pour ne pas continuer la chaîne.
- **Toujours `req.pipe(busboy)`** : sans le pipe, aucun event n'est émis — oubli classique.
- **Drainer les streams rejetés** (`stream.resume()`) : sinon la requête reste suspendue et le client attend indéfiniment.
- `limits.fileSize` : busboy coupe le stream à la limite et pose `file.truncated = true` — dans ce pattern on contrôle la taille dans le driver (plus explicite), les deux approches sont valides.
- Le **Content-Type du multipart** est posé par le navigateur avec son boundary — ne jamais le réécrire côté front.
- Validation MIME côté back obligatoire : le front est contournable (curl, Postman).
- Pas de validation de l'extension seule : toujours le MIME annoncé + idéalement une détection de contenu (magic bytes) pour les uploads sensibles.
- Un seul fichier ici ; pour un lot, cumuler les `uploadPromise` dans un tableau et `Promise.all` dans `finish`.
- Si des secrets interviennent (clés MinIO), ils viennent de `.env` — jamais en dur (voir docker-secrets-env.md).

## État

**vivant** — pattern busboy streaming + délégation au driver via Strategy (voir [upload-strategy.md](upload-strategy.md)) et client MinIO de [minio-storage-express.md](minio-storage-express.md).
