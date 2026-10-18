# CORS — API Express et apps multipage

Stack: Node/Express (ESM), API REST + éventuel front hébergé séparément.
Usage: config CORS pour une API Express consommée par un front sur une autre origine (SPA, Pug hébergé ailleurs), et le cas "pas de CORS nécessaire" en same-origin.

## Règle de décision

- **Same-origin** (front servi par la même app Express : Pug multipage rendu par le serveur, ou static folder) → **pas de CORS du tout**. Le navigateur ne bloque pas same-origin ; ajouter cors ici = complexité inutile.
- **Cross-origin** (API sur api.exemple.com, front sur exemple.com, ou front local Vite sur :5173 qui appelle :3000 en dev) → `cors` middleware explicite, jamais `Access-Control-Allow-Origin: *` en prod avec credentials.

## Config type (API cross-origin)

```js
import 'dotenv/config';
import express from 'express';
import cors from 'cors';

const app = express();
app.use(express.json());

app.use(cors({
  origin: process.env.CORS_ORIGINS?.split(',') ?? ['http://localhost:5173'],
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'g-recaptcha-response'],
}));
```

Points critiques :

- `origin` depuis `process.env` (liste séparée par virgules), jamais de domaine en dur.
- `credentials: true` seulement si cookies cross-origin nécessaires (alors jamais `*` en origin).
- `allowedHeaders` inclut les headers custom (ex. `g-recaptcha-response`, voir fiche recaptcha-siteverify) sinon preflight 403.
- Monté AVANT les routes, juste après `express.json()`.
- Router dev/prod via env : localhost en dev, domaine réel en prod, sans changer le code.

## Multipage same-origin (sites vitrines Pug)

Site vitrine classique (fiche stacks.md) : Express rend les pages Pug et reçoit les POST de formulaires → **same-origin, pas de cors**. La seule exception : formulaires envoyés en fetch vers une API sur un autre domaine — dans ce cas c'est l'API qui porte la config, pas le site.

## Anti-patterns

- `app.use(cors())` sans config : par défaut `origin: *` → échec silencieux dès qu'un header custom ou credentials entre en jeu.
- Config CORS côté front : ça ne fait rien, CORS se configure côté serveur.
- Ajouter cors "au cas où" sur une app same-origin.

## État

**vivant** — validé 2026-10, à instancier au premier projet réel cross-origin.
