# minio-storage-express — Stockage S3 MinIO depuis Express

## Contexte d'usage
- Backend Express (ESM) qui reçoit des fichiers et les stocke dans MinIO.
- Bucket public en lecture (URLs directement affichables navigateur), écriture via le client.
- Parsing multipart avec **Busboy** (stream, pas multer) + validation MIME + limite taille.

## Stack
Node 20+ / Express / minio (sdk) / busboy

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
- `bucketExists()` puis `makeBucket()` si absent.
- Appliquer une **policy publique en lecture** (`s3:GetObject` sur `arn:aws:s3:::<bucket>/*`) pour servir les fichiers directement depuis le navigateur.
- Appelée dans le `startServer()` asynchrone, **avant** `app.listen()` — échec = `process.exit(1)` (fail fast).

## Upload d'un Buffer
```js
await minioClient.putObject(bucket, objectName, buffer, buffer.length, {
    'Content-Type': mimeType,
})
```
Retourner `<bucket>/<objectName>` comme URL logique stockée en base.

## Pattern upload route
Voir le code projet : parsing multipart Busboy, validation MIME, upload, persistance en base.

## Console web MinIO
Accessible directement depuis le navigateur : `http://localhost:9001` — consulter/gérer buckets et objets via l'UI. Connexion avec `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD`.

## Nommage des clés
`<timestamp>-<baseFilename>` et `<timestamp>-<baseFilename>-page-<n>.webp` — évite les collisions sans UUID.

## Notes
- `MINIO_ENDPOINT = minio` = le **nom du service docker-compose** sert de hostname (réseau interne).
- Ne pas commiter les secrets : `.env` — stocker `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD` dans une **note sécurisée macOS** (app Mots de passe / Notes verrouillées).
- `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD` sont aussi les identifiants de la **console web MinIO** (http://localhost:9001).

## État
vivant
