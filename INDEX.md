# INDEX — wiki-llm-code

Point d'entrée pour un LLM : charger INDEX.md, puis seulement les fiches utiles à la tâche.

| Fiche | Stack | Usage | État |
|---|---|---|---|
| [backend/nodejs-express-env.md](backend/nodejs-express-env.md) | Node/Express ESM + dotenv | Règle 0 ESM : `type: module` dans package.json, import/export + extensions .js, __dirname via import.meta.url ; dotenv en 1re ligne, .env non commité, express.json(), fail fast | vivant |
| [backend/express-error-handling.md](backend/express-error-handling.md) | Node/Express (ESM) | Erreurs API : classe `AppError` (statut + stack propre), middleware `errorHandler` monté en DERNIER (4 args obligatoires), 400 JSON / dev verbeux filtré / prod sobre sans fuite | vivant |
| [backend/minio-storage-express.md](backend/minio-storage-express.md) | Node/Express + minio sdk + busboy | Stockage S3 MinIO : client singleton, init bucket idempotent + policy publique, upload Buffer, pattern route upload | vivant |
| [backend/recaptcha-siteverify.md](backend/recaptcha-siteverify.md) | Node/Express + GCP Cloud Functions | reCAPTCHA : vérification serveur du token (header `g-recaptcha-response` → siteverify) | vivant |
| [backend/upload-file-busboy.md](backend/upload-file-busboy.md) | Node/Express + busboy | Upload multipart : middleware uploadValidator (Content-Type), streaming busboy (req.pipe, drain des streams, limits fileSize), validation type/taille serveur, route pattern | vivant |
| [front/recaptcha-invisible-v3.md](front/recaptcha-invisible-v3.md) | Pug + JS natif | reCAPTCHA v3 invisible : chargement script + `grecaptcha.execute` + token en header | vivant |
| [front/upload-file.md](front/upload-file.md) | JS natif / Vue 3 + fetch + FormData | Upload front : input type file, validation type/taille client, FormData multipart (jamais de Content-Type manuel) | vivant |
| [front/daisyui-components.md](front/daisyui-components.md) | Tailwind + DaisyUI + Vue 3 | Inventaire des classes DaisyUI des projets : composants DaisyUI pour les composants (`file-input`, `menu`, modificateurs) + Tailwind pour les utilitaires ; markup type par composant | vivant |
| [front/vuejs-useapi.md](front/vuejs-useapi.md) | Vue 3 Composition API + fetch natif | Composable useAPI : tous les appels API via un seul composable (états loading/error, 400 JSON → message affiché, 500/fallback générique, FormData sans header, pas d'axios) | vivant |
| [front/vuejs-router-menu.md](front/vuejs-router-menu.md) | Vue 3 + vue-router 4 + Tailwind/DaisyUI + FA | Routing : layout + children, menu `menus[]` DaisyUI, linkActiveClass menu-active, router-view v-slot + transition slide-fade (:key, une seule racine par page) | vivant |
| [ide/webstorm-http-upload.md](ide/webstorm-http-upload.md) | WebStorm HTTP Client (.http) | Tester un upload multipart dans l'IDE : boundary + body manuel, inclusion binaire `< ../file.png`, champs texte, env `{{host}}`/tokens non commités, doc de test vivante | vivant |
| [ide/webstorm-ai-mcp.md](ide/webstorm-ai-mcp.md) | WebStorm AI Assistant/Junie + GitHub MCP | Brancher wiki-llm-code dans l'IDE via MCP : fine-grained token Contents Read-only, config mcpServers (clé non vide, Bearer header), Project level, `.junie/guidelines.md`, workflow prompt type | vivant |
| [infra/gcp-secrets.md](infra/gcp-secrets.md) | GCP Secret Manager + Cloud Functions | Créer et monter un secret (dotenv en local, `secrets:` en prod) | vivant |
| [infra/gcp-cloud-functions-checklist.md](infra/gcp-cloud-functions-checklist.md) | GCP Cloud Functions | Checklist création : permissions allUsers, région, secrets, test | vivant |
| [infra/docker-minio-compose.md](infra/docker-minio-compose.md) | Docker Compose + MinIO | Service MinIO en compose : ports 9000/9001, UI navigateur, volume externe (external: true), hostname = nom du service, .env | vivant |
| [infra/docker-secrets-env.md](infra/docker-secrets-env.md) | Docker / Compose / Node | Secrets et .env : périmètre d'exposition (inspect, layers image), checklist .dockerignore/secrets compose/user MinIO dédié | vivant |
| [patterns/express-strategy.md](patterns/express-strategy.md) | Node/Express (protos) | Style délibéré proto : toute la logique dans la route (pas de controller/service), try/catch + next(err) systématique, AppError pour erreurs attendues, signaux d'extraction | vivant |
| [patterns/recaptcha-choix-offre.md](patterns/recaptcha-choix-offre.md) | — | Décision : reCAPTCHA v3 classique vs Enterprise/Fraud Defense | vivant |
| [patterns/upload-strategy.md](patterns/upload-strategy.md) | Strategy pattern (Node) | Drivers de stockage interchangeables : contrat `put`/`delete`, drivers minio/local, factory `UPLOAD_DRIVER` par env, Open/Closed | vivant |
| [stacks.md](stacks.md) | — | Stack de référence + stacks abandonnées | vivant |
| [_archive/recaptcha-v2-v3.md](_archive/recaptcha-v2-v3.md) | Vue 3 générique | Ancienne fiche générique remplacée par le découpage par concern | obsolete |

<!-- Ajouter chaque nouvelle fiche ici : une ligne, stack, usage, état -->
