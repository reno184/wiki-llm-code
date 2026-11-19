# wiki-minio

Dans cette fiche: 2 sections

1. Docker
2. Nodejs javascript

> Minio est toujours utilisé via docker 


## Piège MinIO : les credentials naissent dans le compose, le backend doit lire les mêmes clés

Les credentials MinIO sont créés par le service compose qui crée le user root à partir des variables d'env au premier démarrage.

```yaml
minio:
  image: cgr.dev/chainguard/minio:latest
  restart: always
  user: root
  ports:
    - "9000:9000"
    - "9001:9001"
  environment:
    MINIO_ROOT_USER: ${MINIO_ROOT_USER}
    MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
```

Donc attention Le backend Node (client minio) doit pointer sur **les mêmes valeurs** :

```js
export const minioClient = new Minio.Client({
  endPoint: '127.0.0.1',
  port: 9000,
  useSSL: false,
  accessKey: process.env.MINIO_ROOT_USER,
  secretKey: process.env.MINIO_ROOT_PASSWORD,
})
```

Conséquence : **deux `.env` contenant les mêmes clés** — un à la racine du projet et un côté backend (lu par Node au démarrage).

Notes :

- L'image chainguard tourne en non-root par défaut, d'où le `user: root` : nécessaire pour que MinIO puisse écrire dans son volume.
- `endPoint: '127.0.0.1'` ne marche que si le backend tourne **sur l'hôte** (dev). Backend conteneurisé : utiliser le nom du service (`minio`) — cf. docker-commandes.md.

### todo

MinIO : ne pas utiliser les creds root dans l'app. Créer un user de service dédié borné à un bucket

# minio et nodejs

---
## Backend nodejs

> il faut installer minio pour le client

```bash
npm install minio
```
## Exemple d'upload (PDF + conversion WebP + Mongoose)

```js

export const uploadToMinio = async (bucketName, objectName, buffer, mimeType) => {
    await minioClient.putObject(bucketName, objectName, buffer, buffer.length, {
        'Content-Type': mimeType,
    })

    return `${bucketName}/${objectName}`
}

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

## Chemin d'objet : toujours un UID + nom original en métadonnée

Jamais de filename (espaces, accents) dans la clé MinIO : le chemin est **toujours un UID**, le nom affichable voyage en métadonnée.

1. Réparer le mojibake du multipart — le filename arrive encodé latin1, un nom avec accents (`Rapport été.pdf`) est corrompu à la réception :

```js
const cleanFilename = Buffer.from(uploadedFile.filename, 'latin1').toString('utf8')
```

2. Clé d'objet = UID, rien d'autre :

```js
const objectName = crypto.randomUUID()
```

3. Upload avec le nom original en métadonnée **encodée** — un header de métadonnée doit être ASCII, d'où le `encodeURIComponent` :

```js
await minioClient.putObject(BUCKET_NAME, objectName, buffer, buffer.length, {
    'Content-Type': mimeType,
    'X-Amz-Meta-filename': encodeURIComponent(cleanFilename),
})
```

4. Lecture du nom original (`statObject`) — le fichier est autoporteur, récupérable sans passer par Mongoose :

```js
const stat = await minioClient.statObject(BUCKET_NAME, objectName)
const originalName = decodeURIComponent(stat.metaData['filename']) // → "Rapport été.pdf"
```

⚠️ **Balises de métadonnées : kebab-case, minuscule, mots séparés par des traits d'union.** MinIO n'aime pas le camelCase dans les balises meta — utiliser `total-page`, jamais `totalPage` (la balise est ignorée/corrompue à la lecture dans `stat.metaData`). Toutes les balises suivent le même gabarit :

```js
'X-Amz-Meta-total-page': pages.length,
'X-Amz-Meta-filename': encodeURIComponent(cleanFilename),
```

## statObject est asynchrone — dans une boucle, encapsuler dans un Promise.all

`statObject` renvoie une promesse : dans une boucle sur plusieurs objets (ex. lister les pages d'un PDF et lire leurs métadonnées), on mappe chaque objet en un `async` qui fait le `await` individuel, puis on déroule le tout en parallèle avec `Promise.all`. Jamais de `for...of` avec `await` séquentiel : n appels réseau à la suite au lieu de n appels en parallèle.

```js
Promise.all(
    pageObjects.map(async (pageObj) => {
        const pageStat = await minioClient.statObject(BUCKET_NAME, pageObj.name)
        return {
            page: parseInt(pageStat.metaData['page']),
            url: `/${BUCKET_NAME}/${pageObj.name}`
        }
    })
)
```

Note : les métadonnées arrivent en string (`parseInt` pour `page`), et le nom de balise reste en kebab-case.

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

