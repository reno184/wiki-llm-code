# minio-storage-express — Stockage S3 MinIO depuis Express

## Contexte d'usage
- Backend Express (ESM) qui reçoit des fichiers et les stocke dans MinIO.
- Bucket public en lecture : URLs directement affichables depuis le navigateur.
- MinIO containerisé (Docker Compose), données persistées dans un volume Docker externe.
- Suppression du storage **déclenchée depuis le schéma Mongoose** (hook `deleteOne`) : supprimer le document = supprimer ses fichiers MinIO.

## Stack
Node 20+ / Express / minio (sdk) / busboy / mongoose / docker

## Install
```bash
npm install minio
```

## Config client (singleton)
```js
// config/minio.js
import * as Minio from 'minio'

export const minioClient = new Minio.Client({
    endPoint: process.env.MINIO_ENDPOINT || 'minio', // nom du service Docker
    port: parseInt(process.env.MINIO_PORT, 10) || 9000,
    useSSL: false,
    accessKey: process.env.MINIO_ROOT_USER,
    secretKey: process.env.MINIO_ROOT_PASSWORD,
})
```

## Init du bucket au démarrage (idempotent)
> Pas de try/catch ici : une erreur remonte au `startServer()` qui fait `process.exit(1)` (fail fast). Un bucket absent ne doit jamais laisser le serveur démarrer.

```js
// config/minio.js
export const BUCKET_NAME = 'pdf-thumbnails'

export const initBucket = async (bucketName) => {
    const exists = await minioClient.bucketExists(bucketName)
    if (!exists) {
        await minioClient.makeBucket(bucketName)
        console.log(`Bucket '${bucketName}' créé avec succès.`)
    }

    // Politique rendant les fichiers lisibles publiquement depuis le navigateur
    const policy = {
        Version: '2012-10-17',
        Statement: [
            {
                Effect: 'Allow',
                Principal: {AWS: ['*']},
                Action: ['s3:GetObject'],
                Resource: [`arn:aws:s3:::${bucketName}/*`],
            },
        ],
    }
    await minioClient.setBucketPolicy(bucketName, JSON.stringify(policy))
}
```

Appel dans `index.js`, **avant** `app.listen()` :
```js
const startServer = async () => {
    try {
        await connectDB()
        await initBucket('pdf-thumbnails')

        const PORT = process.env.PORT || 3000
        app.listen(PORT, '0.0.0.0', () => {
            console.log(`Serveur backend démarré sur le port ${PORT}`)
        })
    } catch (error) {
        console.error('❌ Échec du démarrage du serveur :', error)
        process.exit(1)
    }
}
```

## Manager upload (retourne l'URL logique `<bucket>/<objectName>`)
```js
export const uploadToMinio = async (bucketName, objectName, buffer, mimeType) => {
    await minioClient.putObject(bucketName, objectName, buffer, buffer.length, {
        'Content-Type': mimeType,
    })
    return `${bucketName}/${objectName}`
}
```

## Appel dans une route (pattern)
> Rester simple, pas de fallback en cas d'échec d'écriture : l'erreur remonte au `next(err)` global.
```js
const pdfUrl = await uploadToMinio(BUCKET_NAME, pdfKey, fileData.buffer, fileData.mimeType)
// ... traitements dérivés éventuels, puis persistance en base (clé + URL logique)
```

## Delete du storage déclenché par le schéma Mongoose (hook `deleteOne`)
> Intéressant : supprimer le document (ex. `await doc.deleteOne()`) supprime automatiquement le PDF + toutes ses images dans MinIO. Pas d'orphelins dans le storage.
- `{document: true, query: false}` : le hook reçoit le document (`this.pdfKey`), pas la query (`Model.deleteOne()` ne le déclenche pas).
- Import dynamique dans le hook : évite les **dépendances circulaires** (modèle ↔ config minio).
- `removeObjects` pour un lot de clés, `removeObject` pour un seul objet.
- `throw err` : l'échec de suppression MinIO bloque le delete (le doc n'est pas supprimé silencieusement en laissant des fichiers orphelins).

```js
// Hook déclenché par doc.deleteOne() — supprime le PDF et TOUTES ses images dans MinIO
pdfDocumentSchema.pre('deleteOne', {document: true, query: false}, async function () {
    try {
        // Import dynamique : évite les dépendances circulaires
        const {minioClient, BUCKET_NAME} = await import('../config/minio.js')

        if (this.pdfKey) {
            await minioClient.removeObject(BUCKET_NAME, this.pdfKey)
        }

        if (this.images?.length > 0) {
            const keysToDelete = this.images.map(img => img.key).filter(Boolean)
            if (keysToDelete.length > 0) {
                await minioClient.removeObjects(BUCKET_NAME, keysToDelete)
            }
        }
    } catch (err) {
        console.error(`Erreur suppression MinIO pour ${this._id}:`, err)
        throw err
    }
})
```

## Console web MinIO
Accessible directement depuis le navigateur : `http://localhost:9001` — consulter/gérer buckets et objets via l'UI. Connexion avec `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD`.

## Nommage des clés
`<timestamp>-<baseFilename>` et `<timestamp>-<baseFilename>-page-<n>.webp` — évite les collisions sans UUID.

## Notes
- `MINIO_ENDPOINT = minio` = le **nom du service docker-compose** sert de hostname (réseau interne).
- Ne pas commiter les secrets : `.env` — stocker `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD` dans une **note sécurisée macOS** (app Mots de passe / Notes verrouillées).
- `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD` sont aussi les identifiants de la **console web MinIO** (http://localhost:9001).

Example `.env` (valeurs à remplacer par des identifiants forts) :
```
MINIO_ROOT_USER=<user-fort>
MINIO_ROOT_PASSWORD=<mot-de-passe-fort>
```

## État
vivant
