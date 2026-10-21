# wiki-minio-pdf — MinIO : upload/suppression PDF + images, Vue mensuelle

Usage: snippet backend MinIO complet (upload PDF + conversion WebP + hook suppression Mongoose + docker compose). Lecture ~1×/mois, copié/collé manuel.

## Install

```bash
npm install minio
```

## Config client (singleton). Dans backend/config

```js
import * as Minio from 'minio'

export const minioClient = new Minio.Client({
    endPoint: process.env.MINIO_ENDPOINT || 'minio',
    port: parseInt(process.env.MINIO_PORT, 10) || 9000,
    useSSL: false,
    accessKey: process.env.MINIO_ROOT_USER,
    secretKey: process.env.MINIO_ROOT_PASSWORD,
})
```

## Démarrage — après la base mongodb

Appel dans `index.js`, **avant** `app.listen()` :

```js
const startServer = async () => {

        await connectDB()
        await initBucket('pdf-thumbnails')

}
```

## Exemple d'upload (PDF + conversion WebP + Mongoose)

```js
const handlePdfUpload = ({limitMo, firstPageOnly}) => async (req, res, next) => {
    try {
        const fileData = await parseSinglePdf(req, limitMo)

        const timestamp = Date.now()
        const baseFilename = fileData.filename.split('.').slice(0, -1).join('.')

        const pdfKey = `${timestamp}-${fileData.filename}`
        const pdfUrl = await uploadToMinio(BUCKET_NAME, pdfKey, fileData.buffer, fileData.mimeType)

        let webpResult
        try {
            webpResult = await convertPdfToWebp(fileData.buffer, {firstPageOnly})
        } catch (err) {
            throw new AppError(`Erreur conversion PDF: ${err.message}`, 500)
        }

        let imagesData = []

        if (firstPageOnly) {
            const webpKey = `${timestamp}-${baseFilename}-thumb.webp`
            const imageUrl = await uploadToMinio(BUCKET_NAME, webpKey, webpResult, 'image/webp')

            imagesData.push({
                page: 1,
                key: webpKey,
                src: imageUrl
            })
        } else {
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

## Suppression — hook Mongoose deleteOne

```js
pdfDocumentSchema.pre('deleteOne', {document: true, query: false}, async function () {
    try {
        const {minioClient, BUCKET_NAME} = await import('../config/minio.js')

        if (this.pdfKey) {
            await minioClient.removeObject(BUCKET_NAME, this.pdfKey)
        }

        if (this.images && this.images.length > 0) {
            const keysToDelete = this.images.map(img => img.key).filter(Boolean)
            if (keysToDelete.length > 0) {
                await minioClient.removeObjects(BUCKET_NAME, keysToDelete)
            }
        }
    } catch (err) {
        console.error(`Erreur lors de la suppression MinIO pour ${this._id}:`, err)
        throw err
    }
})
```

## Lister les objets d'un bucket — listObjects (renvoyer une liste au front)

MinIO expose `listObjects` : récupère la liste des objets d'un bucket (stream) pour la renvoyer au front. Pas besoin de synchroniser ces données dans Mongoose. Cas idéal : liste simple, non évolutive (pas de filtres complexes, pas de pagination).

```js
const listBucketObjects = (bucketName) => async (req, res, next) => {
    try {
        const objects = []

        const stream = minioClient.listObjects(bucketName, '', true)

        await new Promise((resolve, reject) => {
            stream.on('data', (obj) => {
                objects.push({
                    key: obj.name,
                    size: obj.size,
                    lastModified: obj.lastModified
                })
            })
            stream.on('end', resolve)
            stream.on('error', reject)
        })

        return res.status(200).json({objects})

    } catch (err) {
        next(err)
    }
}
```

## Docker compose

```yaml
  minio:
    image: minio/minio:latest
    restart: always
    ports:
      - "9000:9000"
      - "9001:9001"
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
      MINIO_PUBLIC_URL: http://localhost:9000
    volumes:
      - minio_data:/data
    command: server /data --console-address ":9001"

volumes:
  minio_data:
    external: true
```
## État

**vivant** — snippet brut Renaud (oct. 2026), sans amélioration.
