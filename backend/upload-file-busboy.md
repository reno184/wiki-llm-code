# Upload de fichier — backend (Express + busboy, streaming multipart)

> Miroir local de [backend/upload-file-busboy.md](https://github.com/reno184/wiki-llm-code/blob/main/backend/upload-file-busboy.md) — source de vérité : repo GitHub wiki-llm-code.

Stack: Node 20+ / Express (ESM) / busboy.
Usage: recevoir un upload multipart/form-data dans une route Express, valider le Content-Type via middleware `uploadValidator` dans le dossier `/middlewares`. Le **stockage** du fichier n'est pas le sujet de cette fiche : il est délégué — voir la fiche Strategy [../patterns/upload-strategy.md](../patterns/upload-strategy.md) pour les drivers interchangeables. Fiche back du trio upload fullstack (voir [../front/upload-file.md](../front/upload-file.md)).

```bash
npm install busboy
npm install joi
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

### Vérification des types autorisés (erreur 500)

Le type de fichier accepté est fixé **explicitement côté front** (input `accept`) : une erreur de type est donc improbable → 500.

```js
const ALLOWED_TYPES = ['application/pdf', 'image/jpeg', 'image/png', 'image/webp']

busboy.on('file', (fieldName, file, { filename, mimeType }) => {
    if (!ALLOWED_TYPES.includes(mimeType)) {
        file.resume()
        reject(new AppError(`Type non autorisé : ${mimeType}`, 500))
        return
    }

    const chunks = []

    file.on('data', (chunk) => chunks.push(chunk))

    file.on('end', () => uploadedFile = { filename, buffer: Buffer.concat(chunks), mimeType })
})
```

### Vérification du poids du fichier (erreur 400)

On renvoie une erreur **400** : le poids n'est pas contrôlé côté front (l'utilisateur peut soumettre un fichier trop gros, c'est probable), et une 400 déclenche une popover d'alerte côté front. Une 500 serait traitée comme un bug serveur.

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

### Contrôle un seul fichier autorisé (erreur 500)

Fichier unique ou multiple est fixé **explicitement côté front** (attribut `multiple` ou non) : l'erreur est improbable → 500.

```js
busboy.on('file', (fieldName, file, { filename, mimeType }) => {
    if (uploadedFile !== null) {
        validationError = new AppError('Un seul fichier autorisé', 500)
        file.resume()
        return
    }

    const chunks = []

    file.on('data', (chunk) => chunks.push(chunk))

    file.on('end', () => uploadedFile = { filename, buffer: Buffer.concat(chunks), mimeType })
})
```

### Récupération des champs supplémentaires (`on field`)

Permet de récupérer la valeur de champs texte du formulaire envoyés avec le fichier.

```js
const fields = {}

busboy.on('field', (name, value) => { fields[name] = value })
```

### Optionnel — validation Joi des fields (erreur 400)

Contrôle la validité des champs avec un schema npm Joi. Un champ de formulaire mal rempli est une erreur **probable** (saisie utilisateur) → 400.

⚠️ La validation s'exécute dans le `busboy.on('finish')` — quand **tous** les champs sont reçus. Pas à côté du `on('field')`, sinon on valide un objet à moitié rempli.

```js
import Joi from 'joi'

busboy.on('finish', () => {
    const { error } = Joi.object({
        title: Joi.string().required(),
        author: Joi.string().required()
    }).validate(fields)

    if (error) {
        reject(new AppError(error.details[0].message, 400))
        return
    }

    resolve(uploadedFile)
})
```

### Optionnel — extraire des métadonnées du nom de fichier (patterns regex)

Très optionnel : contrôler le nom du ou des fichiers uploadés et en extraire des infos (une date, un numéro de page, le nom de la feuille d'un Excel...). Pratique pour trier/classer les uploads sans demander de champs supplémentaires au front.

Principe : un dictionnaire de patterns — chaque entrée a sa regex, un exemple, et une fonction `extractMetadata` qui transforme les groupes de capture en métadonnées exploitables.

```js
// utils/filenamePatterns.js
export const FILENAME_PATTERNS = {
    DATE_PREFIX: {
        pattern: /^(\d{4}-\d{2}-\d{2})_[\w-]+$/i,
        example: '2026-06-12_vocabulary.csv',
        extractMetadata: (match) => {
            return {
                uploadDate: match[1]
            }
        }
    },
    PAGE_NUMBER: {
        pattern: /_page_(\d+)(?:\.\w+)?$/i,
        example: 'document_page_42.pdf',
        extractMetadata: (match) => {
            return {
                page: parseInt(match[1], 10)
            }
        }
    },
    SHEET_NAME: {
        pattern: /^([\w-]+)_([\w-]+)$/i,
        example: 'vocabulary_Sheet1.xlsx',
        extractMetadata: (match) => {
            return {
                fileName: match[1],
                sheetName: match[2]
            }
        }
    }
}
```

Utilisation sur le fichier reçu dans la route busboy :

```js
const extractFilenameMetadata = (filename) => {
    for (const [key, def] of Object.entries(FILENAME_PATTERNS)) {
        const match = filename.match(def.pattern)

        if (match) {
            return { pattern: key, metadata: def.extractMetadata(match) }
        }
    }

    return null
}

const metadata = extractFilenameMetadata(fileData.filename)

if (!metadata) {
    throw new AppError(`Nom de fichier non reconnu. Exemple attendu : ${FILENAME_PATTERNS.SHEET_NAME.example}`, 400)
}

const { sheetName } = metadata.metadata
```

Exemples de noms reconnus :

| Fichier uploadé | Pattern détecté | Métadonnées extraites |
|---|---|---|
| `2026-06-12_vocabulary.csv` | DATE_PREFIX | `{ uploadDate: '2026-06-12' }` |
| `document_page_42.pdf` | PAGE_NUMBER | `{ page: 42 }` |
| `vocabulary_Sheet1.xlsx` | SHEET_NAME | `{ fileName: 'vocabulary', sheetName: 'Sheet1' }` |

Notes :

- Un nom qui ne matche **aucun** pattern = erreur 400 (l'utilisateur a nommé son fichier n'importe comment, erreur probable) — à la porte d'entrée, sinon les métadonnées sont `undefined` en aval.
- L'ordre du dictionnaire compte : `Object.entries` teste dans l'ordre d'écriture, le premier match gagne. `DATE_PREFIX` avant `SHEET_NAME`, sinon `2026-06-12_vocabulary` serait interprété comme `fileName: '2026', sheetName: '06-12'`.
- Ajouter un nouveau format = une entrée dans `FILENAME_PATTERNS`, rien d'autre.
- Côté front, **communiquer le format attendu** (placeholder de l'input, message d'aide) : l'API et l'utilisateur doivent parler le même langage de nommage.

## Notes

- **Drainer les streams rejetés** (`file.resume()`) : sinon la requête reste suspendue et le client attend indéfiniment.
- `limits.fileSize` : busboy coupe le stream à la limite et déclenche l'event `limit`.
- Un seul fichier ici ; pour un lot, cumuler les promesses dans un tableau et `Promise.all` dans `finish`.
- Erreurs : try/catch + `next(err)` → errorHandler global (voir [express-error-handling.md](express-error-handling.md)).

## État

**vivant** — pattern busboy minimal + middleware `uploadValidator` + contrôles optionnels (types, poids, fichier unique, champs + Joi) + délégation du stockage au driver via la fiche Strategy.
