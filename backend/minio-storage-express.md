# minio-storage-express — Stockage S3 MinIO depuis Express

## Contexte d'usage
- Backend Express (ESM) qui reçoit des fichiers et les stocke dans MinIO.
- Bucket public en lecture : URLs directement affichables depuis le navigateur.
- MinIO containerisé (Docker Compose), données persistées dans un volume Docker externe.

## Stack
Node 20+ / Express / minio (sdk) / busboy / docker

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
