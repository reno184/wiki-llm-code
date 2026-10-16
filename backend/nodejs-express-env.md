# Node.js/Express — lire process.env (variables d'environnement)

Stack: Node.js, Express, dotenv.
Usage: fiche de base pour tout back Express — charger les variables d'environnement (secrets, config) depuis un fichier `.env` en local.

## Principe

Node.js expose les variables d'environnement via `process.env.X`. En local, elles ne sont pas chargées automatiquement : il faut le package `dotenv`, qui lit le fichier `.env` au racine du projet et les injecte dans `process.env`.

## Étapes

### 1. Installer dotenv

```bash
npm install dotenv
```

### 2. Charger dotenv **au tout début** du point d'entrée

```js
// app.js (ou index.js / server.js) — PREMIÈRE ligne du fichier
require("dotenv").config();

const express = require('express');
const app = express();

// ... maintenant process.env est peuplé partout
```

> ⚠️ `require("dotenv").config()` doit être exécuté **avant** tout module qui lit `process.env` (clés API, config DB, secrets reCAPTCHA…). Sinon les valeurs sont `undefined` au moment de la lecture. Le mettre en première ligne du point d'entrée règle le problème.

### 3. Créer le fichier `.env` (non commité)

```
# .env — à la racine du projet
RECAPTCHA_LEGAL_COMPLIANCE=<secret api key>
PORT=3000
```

### 4. Lire une variable

```js
const secret = process.env.RECAPTCHA_LEGAL_COMPLIANCE;

if (!secret) {
  throw new Error("Variable RECAPTCHA_LEGAL_COMPLIANCE manquante : vérifier .env (local) ou le montage du secret (prod)");
}
```

## Règles

- **`.env` n'est jamais commité** : l'ajouter à `.gitignore`. Commiter un `.env.example` avec les noms de variables sans valeurs, pour documenter.
- **Aucun secret en dur dans le code** : tout passe par `process.env` (voir [../infra/gcp-secrets.md](../infra/gcp-secrets.md) pour la prod GCP).
- **Fail fast** : si une variable obligatoire manque, jeter une erreur explicite au démarrage plutôt que laisser le back tourner avec `undefined`.
- Prod GCP : pas besoin de dotenv — le secret monté via `secrets:` est déjà exposé comme variable d'environnement. Le même `process.env.X` fonctionne donc en local et en prod sans changer le code.

## Pièges connus

- Oublier `require("dotenv").config()` → toutes les variables valent `undefined`.
- Charger dotenv **après** un module qui lit `process.env` au top-level → mêmes symptômes.
- Renommer une variable dans `.env` mais pas dans le code → toujours vérifier les noms (`RECAPTCHA_LEGAL_COMPLIANCE` ≠ `RECAPTCHA_SECRET`).

## État

**vivant** — fiche de base Node/Express, complément de [../infra/gcp-secrets.md](../infra/gcp-secrets.md) pour la prod.
