# minio-storage-express — Stockage S3 MinIO depuis Express

## Contexte d'usage
- Backend Express (ESM) qui reçoit des fichiers et les stocke dans MinIO.
- Bucket public en lecture (URLs directement affichables navigateur).
- Bucket accessible depuis le navigateur
- Bucket inclus dans une image, et données stockés dans un volume Docker


## Stack
Node 20+ / Express / minio (sdk) / busboy / docker


## install

```npm install minio```


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

## Init du bucket au démarrage (idempotent) index.js
 ```js
 const startServer = async () => {
    try {
        // 1. Connexion à MongoDB
        await connectDB()

        // 2. Initialisation du/des buckets MinIO
        await initBucket('pdf-thumbnails')

        // 3. Lancement de l'écoute du serveur HTTP
        const PORT = process.env.PORT || 3000
        const server = app.listen(PORT, '0.0.0.0', () => {
            console.log(`Serveur backend démarré sur le port ${PORT}`);
        });

        // Timeout d'inactivité des sockets HTTP
        server.setTimeout(4000)

    } catch (error) {
        console.error('❌ Échec du démarrage du serveur :', error)
        process.exit(1)
    }
}
```

## setting de setBucketPolicy (example)

```js 

export const BUCKET_NAME = 'pdf-thumbnails'


export const initBucket = async (bucketName) => {
    try {
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
    } catch (err) {
        console.error('Erreur lors de l’initialisation du bucket MinIO:', err)
    }
}

```

### minio manager (example)

```js
export const uploadToMinio = async (bucketName, objectName, buffer, mimeType) => {
    await minioClient.putObject(bucketName, objectName, buffer, buffer.length, {
        'Content-Type': mimeType,
    })
    
    return `${bucketName}/${objectName}`
}
```

### appel example dans la route (pas de fallback)

> note: reste simple pas de fallback si problème écriture

```js

const handlePdfUpload = ({limitMo, firstPageOnly}) => async (req, res, next) => {
    try {
        const fileData = await parseSinglePdf(req, limitMo)

        const timestamp = Date.now()
        const baseFilename = fileData.filename.split('.').slice(0, -1).join('.')

        // 2. Upload du fichier PDF original dans MinIO
        const pdfKey = `${timestamp}-${fileData.filename}`
        const pdfUrl = await uploadToMinio(BUCKET_NAME, pdfKey, fileData.buffer, fileData.mimeType)

        // 3. Conversion PDF -> WebP (L'erreur est automatiquement attrapée par le catch global)
        let webpResult
        try {
            webpResult = await convertPdfToWebp(fileData.buffer, {firstPageOnly})
        } catch (err) {
            throw new AppError(`Erreur conversion PDF: ${err.message}`, 500)
        }

        let imagesData = []

        // 4. Cas A : Première page uniquement (miniature)
        if (firstPageOnly) {
            const webpKey = `${timestamp}-${baseFilename}-thumb.webp`
            const imageUrl = await uploadToMinio(BUCKET_NAME, webpKey, webpResult, 'image/webp')

            imagesData.push({
                page: 1,
                key: webpKey,
                src: imageUrl
            })
        }
        // 4. Cas B : Toutes les pages
        else {
            imagesData = await Promise.all(
                webpResult.map(async (buffer, index) => {
                    const pageNum = index + 1
                    const webpKey = `${timestamp}-${baseFilename}-page-${pageNum}.webp`
                    const imageUrl = await uploadToMinio(BUCKET_NAME, webpKey, buffer, 'image/webp')

                    return {
                        page: pageNum,
                        key: webpKey,
                        src: imageUrl
                    }
                })
            )
        }

        // 5. Sauvegarde du document dans MongoDB via Mongoose
        const pdfDocument = await PdfDocument.create({
            originalName: fileData.filename,
            pdfKey: pdfKey,
            pdfUrl: pdfUrl,
            images: imagesData
        })

        return res.status(201).json({
            message: 'Document PDF uploadé et enregistré avec succès',
            document: pdfDocument
        })

    } catch (err) {
        next(err)
    }
}


```
## Console web MinIO
Accessible directement depuis le navigateur : `http://localhost:9001` — consulter/gérer buckets et objets via l'UI. Connexion avec `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD`.

## Nommage des clés
`<timestamp>-<baseFilename>` et `<timestamp>-<baseFilename>-page-<n>.webp` — évite les collisions sans UUID.

## Notes
- `MINIO_ENDPOINT = minio` = le **nom du service docker-compose** sert de hostname (réseau interne).
- Ne pas commiter les secrets : `.env` — stocker `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD` dans une **note sécurisée macOS** (app Mots de passe / Notes verrouillées).
- `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD` sont aussi les identifiants de la **console web MinIO** (http://localhost:9001).


example .env

```
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin
MINIO_PUBLIC_URL=http://localhost:9000
```

## État
vivant
