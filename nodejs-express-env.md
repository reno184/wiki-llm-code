# Node.js/Express — ESM (type: module) + process.env

Stack: Node.js (ESM), Express, dotenv.
Usage: fiche de base pour tout back Express — projets en **modules ES (import/export)**, chargement des variables d'environnement depuis un fichier `.env` en local.

## Règle 0 — JavaScript en type module (ESM)

**Les projets Node/Express du wiki sont en ES Modules : `"type": "module"` dans `package.json`.**

```json
{
  "type": "module"
}
```

- `import`/`export` partout, **jamais de `require()`** — sauf exceptions ci-dessous.
- Imports **avec l'extension `.js`** (ex. `import { uploadValidator } from '../middlewares/uploadValidator.js'`) : obligatoire en ESM Node, un `import` sans extension casse en exécution directe.
- `import.meta.url` remplace `__dirname` si besoin du chemin du fichier :

```js
import { fileURLToPath } from 'node:url'
import path from 'node:path'

const __dirname = path.dirname(fileURLToPath(import.meta.url))
```

- `process.env` reste identique en ESM.
- Top-level `await` disponible (aucun besoin d'envelopper dans un `async` main).

**Exception : les applications Node/Express en syntaxe Pug (sites vitrines multipage) restent en CommonJS — `require()` et pas de `"type": "module"`.** Voir stacks.md : la stack sites vitrines (Pug + View Transitions + Firebase) est l'exception historique, ne pas la convertir.



## Pense-bête Express : le middleware JSON

Sans `app.use(express.json())`, `req.body` vaut `undefined` sur toutes les requêtes POST/PUT en JSON — même si le front envoie le bon `Content-Type: application/json`.

```js
import 'dotenv/config'
import express from 'express'

const app = express()

app.use(express.json())

app.use('/api/contact', contactRouter)
```

- `express.json()` doit être monté **avant** les routes qui lisent `req.body`.
- Ne parse **que** les requêtes avec `Content-Type: application/json` (d'où l'obligation côté front d'envoyer ce header — voir mode-chat/code-subject/wiki-recaptcha.md).
- Formulaires HTML natifs : `app.use(express.urlencoded({ extended: true }))`.

## Côté front (Vite / Vue) : `import.meta.env`

Le front n'a **ni dotenv ni `process.env`** : Vite expose les variables via `import.meta.env`, et **seules celles préfixées `VITE_`** sont injectées dans le bundle.

```js
// .env à la racine du front
VITE_API_URL=http://localhost:3000
VITE_RECAPTCHA_SITE_KEY=6L...
```

```js
// utilisation dans un composant/script
const apiUrl = import.meta.env.VITE_API_URL
const siteKey = import.meta.env.VITE_RECAPTCHA_SITE_KEY
```

Règles :
- **Jamais de secret dans le front** : tout ce qui est dans `VITE_*` est public (compilé dans le bundle, visible par quiconque ouvre la console réseau). Clé de site reCAPTCHA = OK (publique par conception) ; secret/mot de passe = back uniquement.
- Types : `import.meta.env.MODE` (development/production), `import.meta.env.DEV` / `PROD` pour brancher des comportements.
- En dev, Vite lit `.env`, `.env.development`, `.env.production` automatiquement.
- Exemple complet d'utilisation front (reCAPTCHA + fetch avec `Content-Type: application/json`) : [../mode-chat/code-subject/wiki-recaptcha.md](../mode-chat/code-subject/wiki-recaptcha.md).

## État

**vivant** — fiche de base Node/Express en ESM (`"type": "module"`). Source : [repo wiki-llm-code](https://github.com/reno184/wiki-llm-code).
