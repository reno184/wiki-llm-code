# GCP Cloud Functions — checklist de création (fonction 2e génération / Cloud Run)

Stack: GCP Cloud Functions (onRequest), Node/Express.
Usage: pense-bête à relire à **chaque création d'une nouvelle Cloud Function** — les points que la console GCP ne fait pas par défaut.

## Checklist de création

### 1. Permissions d'invocation — le piège n°1

Par défaut, une nouvelle Cloud Function **n'est pas accessible publiquement** : les requêtes extérieures renvoient 403.

👉 Dans la console GCP, au moment de la création (ou après, dans l'onglet **Permissions**) :

- Ouvrir **Permissions** de la fonction
- Cocher **"Allow unauthenticated invocations"** / `allUsers` (Cloud Run : principal `allUsers` avec rôle *Cloud Run Invoker*)
- ⚠️ Ne le faire **que** pour des fonctions publiques (API web, formulaires). Une fonction interne/admin doit rester sans accès public.

### 2. Région

- Choisir `europe-west1` (ou la région la plus proche des utilisateurs) — définie à la création.

### 3. Secrets

- Déclarer les secrets applicatifs dans `secrets:` à l'export (voir [gcp-secrets.md](gcp-secrets.md)) :

```js
exports.legalCompliance = onRequest({
  region: "europe-west1",
  secrets: ["RECAPTCHA_LEGAL_COMPLIANCE"]
}, app);
```

### 4. Variables d'environnement

- `process.env.X` fonctionne comme en local (dotenv en local, secret monté en prod — voir [../backend/nodejs-express-env.md](../backend/nodejs-express-env.md)).

### 5. Test après déploiement

- `curl` l'URL de la fonction (onglet "URL" dans la console) : 403 → permissions, 500 → logs Cloud Logging.

## Pièges connus

- **403 sur toutes les requêtes** = permissions d'invocation non cochées (le grand classique).
- Fonction déployée mais secrets `undefined` = `secrets:` non déclaré à l'export.
- CORS : pour une API appelée depuis un navigateur, configurer les headers CORS dans le code Express (pas géré par GCP).

## État

**vivant** — checklist issue de l'expérience du site vitrine legaltech.
