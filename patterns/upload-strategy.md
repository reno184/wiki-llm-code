# Strategy pattern — appliqué au stockage d'uploads (drivers interchangeables)

Stack: pattern de conception (aucune dépendance) — exemple d'implémentation : Node/Express + busboy + MinIO/disque local.
Usage: découpler la route d'upload du mécanisme de stockage. La route parle à une **interface**, jamais à une implémentation. Fiche pattern du trio upload fullstack (voir [../front/upload-file.md](../front/upload-file.md) et [../backend/upload-file-busboy.md](../backend/upload-file-busboy.md)).

## Le problème sans Strategy

```js
// ❌ Couplage dur : la route connaît MinIO, ses imports, ses erreurs
const pdfUrl = await uploadToMinio(BUCKET_NAME, key, buffer, mimeType)
```

- Changer de stockage (MinIO → GCS, S3, disque local en dev) = réécrire les routes.
- Impossible de tester la route sans infrastructure MinIO disponible.

## Le contrat (interface implicite JS)

Tous les drivers exposent **la même signature** :

```js
// put(stream, objectName, mimeType, maxFileSize) → Promise<{ url, key }>
// delete(key) → Promise<void>
```

JS n'a pas d'interfaces natives : le contrat est documenté + les drivers sont interchangeables **s'ils satisfont la même signature**. Un agent de code qui ajoute un driver doit respecter ce contrat — aucune autre contrainte.

## Implémentations

### Driver MinIO (prod) — voir [minio-storage-express.md](../backend/minio-storage-express.md) pour le client singleton

```js
// drivers/minio-driver.js
export const minioDriver = {
  name: 'minio',
  put: async (stream, objectName, mimeType, maxFileSize) => {
    // implémentation streaming — voir fiche backend/upload-file-busboy.md
    // ... retourne { url: `${BUCKET_NAME}/${objectName}`, key: objectName }
  },
  delete: (key) => minioClient.removeObject(BUCKET_NAME, key),
}
```

### Driver disque local (dev, tests)

```js
// drivers/local-driver.js
import { createWriteStream } from 'node:fs'
import { mkdir, unlink } from 'node:fs/promises'
import path from 'node:path'

const ROOT = process.env.LOCAL_UPLOAD_DIR || 'uploads'

export const localDriver = {
  name: 'local',
  put: (stream, objectName, mimeType, maxFileSize) =>
    new Promise((resolve, reject) => {
      mkdir(ROOT, { recursive: true }).then(() => {
        let size = 0
        stream.on('data', (c) => {
          size += c.length
          if (size > maxFileSize) { stream.destroy(); reject(new Error('Fichier trop volumineux.')) }
        })
        stream.on('error', reject)
        const out = createWriteStream(path.join(ROOT, objectName))
        stream.pipe(out)
        out.on('error', reject)
        out.on('finish', () => resolve({ url: `/${ROOT}/${objectName}`, key: objectName }))
      }).catch(reject)
    }),
  delete: (key) => unlink(path.join(ROOT, key)),
}
```

## Sélection du driver — factory par env (pas de if dans les routes)

```js
// drivers/index.js
import { minioDriver } from './minio-driver.js'
import { localDriver } from './local-driver.js'

const drivers = { minio: minioDriver, local: localDriver }

export const getUploadDriver = () => {
  const driver = drivers[process.env.UPLOAD_DRIVER]
  if (!driver) throw new Error(`Driver d'upload inconnu : ${process.env.UPLOAD_DRIVER}`) // fail fast
  return driver
}
```

```bash
# .env
UPLOAD_DRIVER=local   # dev : disque local, pas de Docker/MinIO requis
UPLOAD_DRIVER=minio   # prod : via docker compose
```
```js
// ✅ Style wiki : pas de commentaire, le code se lit de haut en bas
router.post('/api/material', uploadValidator, (req, res, next) => {
  try {
    const busboy = Busboy({ headers: req.headers, limits: { fileSize: MAX_SIZE } })

    const fields = {}

    busboy.on('field', (name, value) => { fields[name] = value })

    busboy.on('file', (name, stream, info) => {
      const objectName = `${Date.now()}-${info.filename}`

      uploadPromise = getUploadDriver().put(stream, objectName, info.mimeType, MAX_SIZE)
    })

    req.pipe(busboy)

    // ...
  } catch (err) {
    next(err)
  }
})
```
## Pourquoi c'est le bon pattern ici (et pas les autres)

- **Open/Closed** (SOLID) : ajouter un driver GCS/S3 = nouveau fichier + une ligne dans `drivers` — **zéro modification des routes**.
- La route d'upload ne connaît que `getUploadDriver().put(...)` : testable avec le driver local, sans infra.
- ≠ Factory seul : la factory **sélectionne**, le Strategy **définit la famille de comportements** interchangeables. Ici on a les deux, chacun dans son rôle.
- ≠ Adapter : on ne convertit pas une API existante, on définit un contrat interne volontairement.

## Notes pour les agents de code (wiki → génération)

- Toute nouvelle cible de stockage = **nouveau driver satisfaisant le contrat** (`put`/`delete`), jamais de `if (storage === 'gcs')` dans une route.
- Le driver est choisi **une fois par requête** via `getUploadDriver()` (lecture env) — pas d'instance globale mutée.
- Même en local, respecter les limites (taille, MIME) : les validations sont dans la route (fiche busboy), pas dans les drivers — un driver stocke, il ne décide pas.
- `.env` non commité (voir [nodejs-express-env.md](../backend/nodejs-express-env.md)).

## État

**vivant** — pattern appliqué à l'upload fullstack ; drivers minio + local, sélection par `UPLOAD_DRIVER`.
