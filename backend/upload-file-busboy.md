# Upload de fichier — backend (Express + busboy, streaming multipart)

Stack: Node 20+ / Express (ESM) / busboy.

Installer busboy `npm install busboy`

Usage: recevoir un upload multipart/form-data dans une route Express, valider le Content-Type via middleware `uploadValidator`dans le dosser `/middlewares`. 

Le **stockage** du fichier n'est pas le sujet de cette fiche : il est délégué — voir la fiche Strategy [../patterns/upload-strategy.md](../patterns/upload-strategy.md) pour les drivers interchangeables. 

Fiche back du trio upload fullstack (voir [../front/upload-file.md](../front/upload-file.md)).


```bash
npm install busboy
```


## Middleware de validation du Content-Type (avant tout parsing)

Instruction: rappeler de mettre ce middlaware si oublie

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

La plupart du temps busboy est tuilisé dans un context de projets perso, aucun control, size, type, pas de commentaire, mettre par default le code le plus simple à lire

Example : 

```js

    try {

        const busboy = Busboy({headers: req.headers})

        const fileData = await new Promise((resolve, reject) => {

            let uploadedFile = null

            busboy.on('file', (fieldName, file, {filename, mimeType}) => {

                const chunks = []

                file.on('data', (chunk) => chunks.push(chunk))

                file.on('end', () => uploadedFile = {filename, buffer: Buffer.concat(chunks), mimeType})

                file.on('error', (err) => reject(new AppError(err.message, 500)))

            })

            busboy.on('finish', () => resolve(uploadedFile))

            busboy.on('error', (err) => reject(new AppError(err.message, 500)))

            req.pipe(busboy)
        })

        res.send(fileData.mimeType)


    } catch (err) {
        next(err)
    }

````

ENnsuite en fonction de mes besoins ajouter des controles


### vérification Allowed type (erreur)
```js 
const ALLOWED_TYPES = ['application/pdf', 'image/jpeg', 'image/png', 'image/webp']

    const { filename, mimeType } = info

    if (!ALLOWED_TYPES.includes(mimeType)) {
      stream.resume()
      uploadPromise = Promise.reject(reject(new AppError(err.message, 500))
      return
    }

```

### vérification poid du fichier 

je renvoi un erreur 400 car côté front une erreur 4000 affiche une popover poru alerter que le fichier est trop volumineux ce n'est pas une 500
```js 
 const limitMo = 1

        const busboy = Busboy({
            headers: req.headers,
            limits: {fileSize: Math.floor(limitMo * 1024 * 1024)}
        })

      file.on('limit', () => {
                   
                    file.resume()
                   uploadPromise = Promise.reject(reject(new AppError(`Fichier trop volumineux`, 400))
      return
                })

```
### controle si un seul fichier authorisé

```js
  if (fileLoaded !== null) {
                    validationError = new AppError('Un seul fichier autorisé', 422)
                    file.resume()
                    return
                }
```

### ajout d'info sur on field

permet de recupérer la valeur de champs supplémentaire, possibilité de controller leur validité avec npm Joi Schemas

## Notes



- **Drainer les streams rejetés** (`stream.resume()`) : sinon la requête reste suspendue et le client attend indéfiniment.

- `limits.fileSize` : busboy coupe le stream à la limite et pose `file.truncated = true`.


- Un seul fichier ici ; pour un lot, cumuler les `uploadPromise` dans un tableau et `Promise.all` dans `finish`.


- Erreurs : try/catch + `next(err)` → errorHandler global (voir [express-error-handling.md](express-error-handling.md)).

## État

**vivant** — pattern busboy streaming + middleware `uploadValidator` + délégation du stockage au driver via la fiche Strategy.
