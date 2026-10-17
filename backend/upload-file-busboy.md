# Upload de fichier — backend (Express + busboy, streaming multipart)

Stack: Node 20+ / Express (ESM) / busboy.
Usage: recevoir un upload multipart/form-data dans une route Express, valider le Content-Type via middleware, streamer le fichier, valider type/taille. Le **stockage** du fichier n'est pas le sujet de cette fiche : il est délégué — voir la fiche Strategy [../patterns/upload-strategy.md](../patterns/upload-strategy.md) pour les drivers interchangeables. Fiche back du trio upload fullstack (voir [../front/upload-file.md](../front/upload-file.md)).

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

  const isMultipart = contentType.includes('multipart/form-data')

  if (!isMultipart) {
    return next(new Error('Expected multipart/form-data'))
  }

  next()
}
```

## Route upload — pattern

Le middleware se branche **dans la chaîne de la route**, avant le handler busboy. Le stockage est délégué au driver choisi par env (fiche Strategy) — cette route ne connaît aucune implémentation de stockage :

```js
import express from 'express'
import Busboy from 'busboy'
import { uploadValidator } from '../middlewares/uploadValidator.js'
import { getUploadDriver } from '../drivers/index.js'

const router = express.Router()

const MAX_SIZE = 20 * 1024 * 1024
const ALLOWED_TYPES = ['application/pdf', 'image/jpeg', 'image/png', 'image/webp']

router.post('/api/upload', uploadValidator, (req, res, next) => {
  const busboy = Busboy({
    headers: req.headers,
    limits: { fileSize: MAX_SIZE },
  })

  const fields = {}
  let uploadPromise = null

  busboy.on('field', (name, value) => { fields[name] = value })

  busboy.on('file', (name, stream, info) => {
    const { filename, mimeType } = info

    if (!ALLOWED_TYPES.includes(mimeType)) {
      stream.resume()
      uploadPromise = Promise.reject(new Error('Type de fichier non autorisé.'))
      return
    }

    const objectName = `${Date.now()}-${filename.replace(/[^a-zA-Z0-9._-]/g, '_')}`

    uploadPromise = getUploadDriver().put(stream, objectName, mimeType, MAX_SIZE)
  })

  busboy.on('error', next)
  busboy.on('finish', async () => {
    try {
      if (!uploadPromise) throw new Error('Aucun fichier reçu.')
      const result = await uploadPromise
      res.status(201).json({ ...result, fields })
    } catch (err) {
      next(err)
    }
  })

  req.pipe(busboy)
})

export default router
```

## Notes

- **`uploadValidator` protège l'entrée de la chaîne** : header absent ou non-multipart → rejet avant tout parsing. Il ne remplace ni les limites de taille (driver) ni la validation MIME par fichier (handler busboy) — chaque couche garde son rôle.
- Un middleware Express = une fonction `(req, res, next)` placée dans la chaîne de la route (`router.post(path, uploadValidator, handler)`) ou en `app.use` global. `next(err)` saute au middleware d'erreur global ; **toujours `return` après `next(err)`** pour ne pas continuer la chaîne.
- **Toujours `req.pipe(busboy)`** : sans le pipe, aucun event n'est émis — oubli classique.
- **Drainer les streams rejetés** (`stream.resume()`) : sinon la requête reste suspendue et le client attend indéfiniment.
- `limits.fileSize` : busboy coupe le stream à la limite et pose `file.truncated = true`.
- Le **Content-Type du multipart** est posé par le navigateur avec son boundary — ne jamais le réécrire côté front.
- Validation MIME côté back obligatoire : le front est contournable (curl, Postman).
- Un seul fichier ici ; pour un lot, cumuler les `uploadPromise` dans un tableau et `Promise.all` dans `finish`.
- Le driver de stockage (signature `put`/`delete`) est défini dans la fiche [../patterns/upload-strategy.md](../patterns/upload-strategy.md) — cette fiche ne documente **aucune implémentation** : la route reste agnostique du stockage.
- Erreurs : try/catch + `next(err)` → errorHandler global (voir [express-error-handling.md](express-error-handling.md)).

## État

**vivant** — pattern busboy streaming + middleware `uploadValidator` + délégation du stockage au driver via la fiche Strategy.
