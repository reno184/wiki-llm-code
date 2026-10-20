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

Le front ne lit **pas `process.env`** : Vite expose les variables via `import.meta.env`. Les variables d'environnement se mettent à jour **au lancement** — deux façons :

### 1. Dans le script `package.json` (la façon courante)

La variable est passée **dans la commande de lancement** du script — préfixée `VITE_` pour être exposée :

```json
{
  "scripts": {
    "dev": "VITE_API_URL=http://localhost:3000 vite",
    "dev:docker": "VITE_API_URL=http://host.docker.internal:3000 vite",
    "build": "VITE_API_URL=https://api.mondomaine.com vite build"
  }
}
```

```js
// dans le code
const apiUrl = import.meta.env.VITE_API_URL
```

- Changer la valeur = changer le script (ou en créer un par contexte), **pas** le code.
- Avantage : chaque contexte (local, docker, prod) a son script explicite — rien de caché dans un fichier.

### 2. Dans `vite.config.js`

Pour les variables **calculées** ou dérivées, via `define` :

```js
import { defineConfig } from 'vite'

export default defineConfig({
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version)
  }
})
```

Ou pour injecter des `VITE_` selon l'environnement de launch :

```js
import { defineConfig, loadEnv } from 'vite'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd(), '')
  return {
    define: {
      __API_URL__: JSON.stringify(env.VITE_API_URL)
    }
  }
})
```

### Règles

- **Préfixe `VITE_` obligatoire** pour qu'une variable passée au lancement soit exposée via `import.meta.env` (tout ce qui n'est pas préfixé reste invisible du code front).
- Vite lit aussi des fichiers `.env` / `.env.development` / `.env.production` à la racine — méthode valable, mais la pratique Renaud = scripts package.json ou vite.config.js.
- **Jamais de secret dans le front** : tout `VITE_*` est compilé dans le bundle = public (visible dans la console réseau). Clé de site reCAPTCHA = OK (publique par conception) ; secret/mot de passe = back uniquement.
- `import.meta.env.MODE` (`development`/`production`), `.DEV` / `.PROD` pour brancher des comportements.
- Exemple complet (reCAPTCHA front + fetch `Content-Type: application/json`) : [wiki-recaptcha.md](wiki-recaptcha.md).

## État

**vivant** — reformulée octobre 2026 (fusion backend dotenv + front Vite, exemples ajoutés).
