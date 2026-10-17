# Gestion d'erreurs Express — AppError + errorHandler (middleware global)

Stack: Node 20+ / Express (ESM), dotenv.
Usage: pattern de gestion d'erreurs centralisé d'une API Express — classe `AppError` pour lever des erreurs avec statut HTTP, middleware `errorHandler` monté en **dernier** qui formate la réponse selon l'environnement (dev verbeux / prod sobre). Complète la fiche [nodejs-express-env.md](nodejs-express-env.md) (fail fast au démarrage) — ici c'est la gestion des erreurs **en cours de requête**.

## Politique de statuts : 400 (probable) vs 500 (improbable)

**Règle : une erreur qu'on peut anticiper côté client = 400. Une erreur improbable = 500. 95 % des erreurs sont des 500.**

- **400** = erreur **probable**, anticipée, qui vient de l'utilisateur : poids du fichier trop gros (popover front), mot de passe incorrect, validation de formulaire. Le front exploite le message.
- **500** = erreur **improbable**, par construction : si le comportement est fixé explicitement côté front, l'erreur ne devrait jamais arriver → c'est un bug, pas un cas utilisateur. Exemples : type de fichier non autorisé (le front a un `accept`), fichier multiple sur un input single (`multiple` absent), 404 métier inattendu.
- Ne pas utiliser 415/422/4xx exotiques : deux statuts suffisent (400/500), sauf décision explicite.
- ⚠️ Si un 422 (ou autre 4xx exotique) apparaît dans le wiki, c'est une erreur à corriger.

```js
throw new AppError('Fichier trop volumineux', 400)      // probable : le front ne contrôle pas le poids
throw new AppError('Mot de passe incorrect', 400)        // probable : erreur de saisie utilisateur
throw new AppError('Type non autorisé', 500)             // improbable : le front fixe accept
throw new AppError('Un seul fichier autorisé', 500)      // improbable : le front fixe multiple ou non
```

## Classe AppError — `utils/AppError.js`

Erreur métier avec statut HTTP, stack propre (filtrée du bruit applicatif).

```js
export class AppError extends Error {
  constructor (message, status) {
    super(message)
    this.status = status
    Error.captureStackTrace(this, this.constructor)
  }
}
```

Usage dans une route/service :

```js
import { AppError } from '../utils/AppError.js'

throw new AppError('Fichier trop volumineux', 400)   // erreur probable → 400
throw new AppError('Type non autorisé', 500)            // erreur improbable → 500
```

- `Error.captureStackTrace(this, this.constructor)` : la stack démarre **à l'endroit du throw**, pas dans le constructeur — le bruit de la classe elle-même est éliminé.
- Sans `AppError`, une erreur standard arrive dans `errorHandler` avec `err.status === undefined` → traitée comme 500. C'est le contrat : **erreur attendue = AppError avec statut, erreur inattendue = 500 générique**.

## Middleware errorHandler — `middlewares/errorHandler.js`

```js
export function errorHandler(err, req, res, _next) {
    if (err.status === 400 || err.status === '400') {
        res.status(400).json({ code: 400, message: err.message })
    } else {
        const isDev = process.env.NODE_ENV !== 'production'
        if (isDev) {
            const fullStack = [
                '=== ERREUR (API) ===',
                `Message: ${err.message}`,
                `Status: ${err.status || 500}`,
                '',
                '=== REQUÊTE (API) ===',
                `Méthode: ${req.method}`,
                `URL: ${req.originalUrl}`,
                '',
                '=== SERVER STACK (API) ===',
                err.stack
                    ?.split('\n')
                    .filter(line => !line.includes('node_modules'))
                    .join('\n')
            ].filter(line => !line.includes('express/lib')).join('\n')

            res.status(err.status || 500).send(fullStack)
        } else {
            // le seul log en production
            console.error('ERROR -- PROD ENV -- 💥:', err) // log critique pour les développeurs
            res.status(500).send('Internal Server Error')
        }
    }
}
```

## Montage dans `index.js` — **après toutes les routes**

```js
import { errorHandler } from './middlewares/errorHandler.js'
import reviewRouter from './routes/reviewRouter.js'

app.use('/api/review', reviewRouter)
// ... toutes les autres routes ...

app.use(errorHandler) // ⚠️ toujours en DERNIER, après chaque app.use de route
```

## Comment une erreur atteint errorHandler

Trois chemins, tous convergent vers le middleware :

1. **Sync** : une exception dans un handler → Express l'attrape automatiquement et saute au middleware d'erreur.
2. **Async** : Express n'attrape PAS les promesses rejetées (avant la v5) → `return next(err)` explicite, ou wrapper async, ou `express-async-errors`.
3. **Volontaire** : `next(new AppError('...', 404))` / `throw new AppError(...)` dans le code métier.

Exemple de bout en bout avec le trio upload :

```js
// route — le rejet du driver/busboy remonte via next(err)
} catch (err) {
  next(err) // → errorHandler → 400 si AppError(…, 400), sinon 500
}
```

## Notes

- **La signature 4 arguments est obligatoire** : `(err, req, res, next)`. Express identifie un middleware d'erreur **par le nombre d'arguments** — avec 3 arguments c'est un middleware normal qui ne recevra jamais d'erreur. Le `_next` souligné : jamais appelé, mais il DOIT être là.
- **L'ordre de montage est critique** : `app.use(errorHandler)` après toutes les routes, sinon il capte les erreurs des routes non encore déclarées... et les routes suivantes deviennent inaccessibles.
- **Branche 400** : les erreurs de validation client renvoient un JSON `{ code, message }` exploitable par le front (affichage direct du message — il vient de nous, contrôlé). Le `err.status === '400'` (string) protège des statuts mal typés.
- **Prod = surface minimale** : jamais de stack, jamais de `err.message` brut en 500 (fuite d'infos internes : chemins, versions, requêtes). Réponse fixe `Internal Server Error` + un seul `console.error` pour les logs. En dev au contraire : tout le contexte (message, requête, stack filtrée).
- La stack dev est **filtrée** : `node_modules` et `express/lib` retirés — ne reste que le code applicatif.
- Le statut final sort de `err.status || 500` : une erreur sans statut = 500 par défaut.
- Ce middleware est le point de convergence cité dans les autres fiches backend ("l'erreur remonte au middleware global") — ex. [minio-storage-express.md](minio-storage-express.md), [upload-file-busboy.md](upload-file-busboy.md).

## État

**vivant** — pattern standard de toutes les API Express du wiki : `AppError` + `errorHandler` monté en dernier, 400 JSON / 500 sobre en prod.
