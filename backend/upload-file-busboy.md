# Upload de fichier — backend (Express + busboy, streaming multipart)

Stack: Node 20+ / Express (ESM) / busboy.
Usage: recevoir un upload multipart/form-data dans une route Express, valider le Content-Type via middleware `uploadValidator` dans le dossier `/middlewares`. Le **stockage** du fichier n'est pas le sujet de cette fiche : il est délégué — voir la fiche Strategy [../patterns/upload-strategy.md](../patterns/upload-strategy.md) pour les drivers interchangeables. Fiche back du trio upload fullstack (voir [../front/upload-file.md](../front/upload-file.md)).

```bash
npm install busboy
```

## Middleware de validation du Content-Type (avant tout parsing)

À rappeler systématiquement sur toute route d'upload : ne jamais lancer busboy sans ce middleware en amont.

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

## Route upload — pattern minimal

La plupart du temps busboy est utilisé dans un contexte de projets perso : aucun contrôle, pas de commentaire, le code le plus simple à lire, de haut en bas.

```js
try {
    const busboy = Busboy({ headers: req.headers })

    const fileData = await new Promise((resolve, reject) => {
        let uploadedFile = null

        busboy.on('file', (fieldName, file, { filename, mimeType }) => {
            const chunks = []

            file.on('data', (chunk) => chunks.push(chunk))

            file.on('end', () => uploadedFile = { filename, buffer: Buffer.concat(chunks), mimeType })

            file.on('error', (err) => reject(new AppError(err.message, 500)))
        })

        busboy.on('finish', () => resolve(uploadedFile))

        busboy.on('error', (err) => reject(new AppError(err.message, 500)))

        req.pipe(busboy)
    })

    res.status(201).send(fileData)

} catch (err) {
    next(err)
}
```

Ensuite, en fonction des besoins, ajouter les contrôles ci-dessous.

### Vérification des types autorisés (erreur 415)

```js
const ALLOWED_TYPES = ['application/pdf', 'image/jpeg', 'image/png', 'image/webp']

busboy.on('file', (fieldName, file, { filename, mimeType }) => {
    if (!ALLOWED_TYPES.includes(mimeType)) {
        file.resume()
        reject(new AppError(`Type non autorisé : ${mimeType}`, 415))
        return
    }

    const chunks = []

    file.on('data', (chunk) => chunks.push(chunk))

    file.on('end', () => uploadedFile = { filename, buffer: Buffer.concat(chunks), mimeType })
})
```

### Vérification du poids du fichier (erreur 400)

On renvoie une erreur **400** et non 500 : côté front, une erreur 400 affiche une popover pour alerter que le fichier est trop volumineux — une 500 serait traitée comme un bug serveur.

```js
const limitMo = 1

const busboy = Busboy({
    headers: req.headers,
    limits: { fileSize: Math.floor(limitMo * 1024 * 1024) }
})

file.on('limit', () => {
    file.resume()
    reject(new AppError('Fichier trop volumineux', 400))
})
```

### Contrôle un seul fichier autorisé (erreur 422)

```js
busboy.on('file', (fieldName, file, { filename, mimeType }) => {
    if (uploadedFile !== null) {
        validationError = new AppError('Un seul fichier autorisé', 422)
        file.resume()
        return
    }

    const chunks = []

    file.on('data', (chunk) => chunks.push(chunk))

    file.on('end', () => uploadedFile = { filename, buffer: Buffer.concat(chunks), mimeType })
})
```

### Ajout d'infos sur `on field`

Permet de récupérer la valeur de champs supplémentaires (champs texte du formulaire envoyés avec le fichier). Possibilité de contrôler leur validité avec les schemas npm Joi.

```js
const fields = {}

busboy.on('field', (name, value) => { fields[name] = value })
```

## Notes

- **Drainer les streams rejetés** (`file.resume()`) : sinon la requête reste suspendue et le client attend indéfiniment.
- `limits.fileSize` : busboy coupe le stream à la limite et déclenche l'event `limit`.
- Un seul fichier ici ; pour un lot, cumuler les promesses dans un tableau et `Promise.all` dans `finish`.
- Erreurs : try/catch + `next(err)` → errorHandler global (voir [express-error-handling.md](express-error-handling.md)).

## État

**vivant** — pattern busboy minimal + middleware `uploadValidator` + contrôles optionnels + délégation du stockage au driver via la fiche Strategy.
