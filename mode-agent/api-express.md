# Stack backend

Node 20+ / Express (ESM), busboy, mongoose.

- Toute la logique métier dans la route : pas de couche controller ni service.
- Route : toujours try/catch, erreurs remontées via `next(err)` — jamais de rejet silencieux. Fallback `errorHandle` dans `index.js`.
- Toujours `AppError` (500), sauf 2 exceptions :
  1. 400 si poids fichier > limite busboy
  2. 400 si email/mot de passe invalide au signIn

## Contrôles dans index.js

- Si `app.use(express.json())` absent → le signaler.