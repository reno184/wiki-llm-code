# Côté backend

## Principe

Node.js expose les variables d'environnement via `process.env.X`. 

En local, elles ne sont pas chargées automatiquement : le package `dotenv` lit le fichier `.env` à la racine et les injecte dans `process.env`.

## Étapes

1. `npm install dotenv`
2. Charger au **tout début** du point d'entrée :

```js
// index.js — PREMIÈRE ligne
import 'dotenv/config'
```

3. `.env` à la racine (non commité),committer un `.env.example` avec noms de variables sans valeurs. :

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

# Côté front

process.env n'est pas lu, il faut utiliser Vite et injecter un vairaible environnement directement par le script dans package.son ou par vite.config.js