# INDEX — wiki-llm-code

Point d'entrée pour un LLM : charger INDEX.md, puis seulement les fiches utiles à la tâche.

| Fiche | Stack | Usage | État |
|---|---|---|---|
| [backend/nodejs-express-env.md](backend/nodejs-express-env.md) | Node/Express + dotenv | Lire process.env : dotenv en 1re ligne, .env non commité, express.json(), fail fast | vivant |
| [backend/recaptcha-siteverify.md](backend/recaptcha-siteverify.md) | Node/Express + GCP Cloud Functions | reCAPTCHA : vérification serveur du token (header `g-recaptcha-response` → siteverify) | vivant |
| [front/recaptcha-invisible-v3.md](front/recaptcha-invisible-v3.md) | Pug + JS natif | reCAPTCHA v3 invisible : chargement script + `grecaptcha.execute` + token en header | vivant |
| [infra/gcp-secrets.md](infra/gcp-secrets.md) | GCP Secret Manager + Cloud Functions | Créer et monter un secret (dotenv en local, `secrets:` en prod) | vivant |
| [infra/gcp-cloud-functions-checklist.md](infra/gcp-cloud-functions-checklist.md) | GCP Cloud Functions | Checklist création : permissions allUsers, région, secrets, test | vivant |
| [patterns/recaptcha-choix-offre.md](patterns/recaptcha-choix-offre.md) | — | Décision : reCAPTCHA v3 classique vs Enterprise/Fraud Defense | vivant |
| [stacks.md](stacks.md) | — | Stack de référence + stacks abandonnées | vivant |
| [_archive/recaptcha-v2-v3.md](_archive/recaptcha-v2-v3.md) | Vue 3 générique | Ancienne fiche générique remplacée par le découpage par concern | obsolete |

<!-- Ajouter chaque nouvelle fiche ici : une ligne, stack, usage, état -->
