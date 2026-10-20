# wiki-variable-env — variables d'environnement : backend dotenv / front Vite

Usage: référence unique pour les variables d'environnement des projets Renaud (Node.js backend + front Vite/Vue). Jamais chargé par un agent. Version complète backend (ESM, fail fast, prod GCP) → [../../backend/nodejs-express-env.md](../../backend/nodejs-express-env.md).

## Côté backend

Node.js expose les variables d'environnement via `process.env.X`.

En local, elles ne sont pas chargées automatiquement : le package `dotenv` lit le fichier `.env` à la racine et les injecte dans `process.env`.

```bash
npm install dotenv
```

1. Charger au **tout début** du point d'entrée :

```js
// index.js — PREMIÈRE ligne
import 'dotenv/config'
```

2. `.env` à la racine (**non commité**, `~/.gitignore`), et committer un `.env.example` avec les noms de variables sans valeurs :

```
RECAPTCHA_LEGAL_COMPLIANCE=<secret api key>
PORT=3000
```

3. Lire avec fail fast :

```js
const secret = process.env.RECAPTCHA_LEGAL_COMPLIANCE
if (!secret) {
  throw new Error('Variable RECAPTCHA_LEGAL_COMPLIANCE manquante : vérifier .env (local) ou le secret monté (prod)')
}
```

## Côté front (Vite / Vue)

Le front ne lit **pas `process.env`** : Vite expose les variables via `import.meta.env`, et **seules celles préfixées `VITE_`** sont injectées dans le bundle. Deux façons de les définir :

**1. Fichiers `.env`** (à la racine du front — Vite lit `.env`, `.env.development`, `.env.production` automatiquement) :

```
VITE_API_URL=http://localhost:3000
VITE_RECAPTCHA_SITE_KEY=6L...
```

```js
const apiUrl = import.meta.env.VITE_API_URL
```

**2. Par le script `package.json`** (utile pour un port ou une URL qui change selon l'environnement) :

```json
{
  "scripts": {
    "dev": "VITE_API_URL=http://localhost:3000 vite",
    "dev:docker": "VITE_API_URL=http://host.docker.internal:3000 vite"
  }
}
```

**3. Par `vite.config.js`** (pour les variables calculées) :

```js
import { defineConfig } from 'vite'

export default defineConfig({
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version)
  }
})
```

**Règles front** :
- **Jamais de secret dans le front** : tout `VITE_*` est compilé dans le bundle = public (visible par quiconque ouvre la console réseau). Clé de site reCAPTCHA = OK (publique par conception) ; secret/mot de passe = back uniquement.
- `import.meta.env.MODE` (`development`/`production`), `.DEV` / `.PROD` pour brancher des comportements.
- Exemple complet (reCAPTCHA front + fetch `Content-Type: application/json`) : [wiki-recaptcha.md](wiki-recaptcha.md).

## État

**vivant** — reformulée octobre 2026 (fusion backend dotenv + front Vite, exemples ajoutés).
