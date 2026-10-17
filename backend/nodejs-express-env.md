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

## Principe

Node.js expose les variables d'environnement via `process.env.X`. En local, elles ne sont pas chargées automatiquement : le package `dotenv` lit le fichier `.env` à la racine et les injecte dans `process.env`.

## Étapes

1. `npm install dotenv`
2. Charger au **tout début** du point d'entrée :

```js
// index.js — PREMIÈRE ligne
import 'dotenv/config'
```

3. `.env` à la racine (non commité) :

```
RECAPTCHA_LEGAL_COMPLIANCE=<secret api key>
PORT=3000
```

4. Lire :

```js
const secret = process.env.RECAPTCHA_LEGAL_COMPLIANCE
if (!secret) {
  throw new Error('Variable RECAPTCHA_LEGAL_COMPLIANCE manquante : vérifier .env (local) ou le secret monté (prod)')
}
```

## Règles

- `.env` jamais commité (`.gitignore`) ; committer un `.env.example` avec noms de variables sans valeurs.
- Aucun secret en dur : tout via `process.env` (prod GCP : voir gcp-secrets.md).
- **Fail fast** : variable obligatoire manquante = erreur explicite au démarrage.
- `import 'dotenv/config'` doit s'exécuter **avant** tout module qui lit `process.env`, sinon valeurs `undefined`.
- Prod GCP : pas besoin de dotenv — secret monté via `secrets:` déjà exposé en variable d'environnement ; même code local/prod.

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
- Ne parse **que** les requêtes avec `Content-Type: application/json` (d'où l'obligation côté front d'envoyer ce header — voir recaptcha-invisible-v3.md).
- Formulaires HTML natifs : `app.use(express.urlencoded({ extended: true }))`.

## État

**vivant** — fiche de base Node/Express en ESM (`"type": "module"`). Source : [repo wiki-llm-code](https://github.com/reno184/wiki-llm-code).
