# Stacks

## Stack actuelle — sites vitrines

- **Back** : Node.js + Express
- **Templating** : Pug (syntaxe Pug pour les vues)
- **Architecture front** : site **multipage** (pas de SPA)
- **Animations** : JavaScript **View Transitions API** — transitions/animations entre les pages
- **Anti-bot** : reCAPTCHA v3 (voir les fiches reCAPTCHA du wiki)
- **Nom de domaine** : Infomaniak
- **Hosting** : Firebase Hosting
- **Backend serverless** : Firebase Cloud Functions
- **Secrets** : dotenv en local, secrets montés en prod (voir [gcp-secrets.md](infra/gcp-secrets.md))

## Stacks abandonnées (ne plus générer de code dessus)

- Angular + Firebase (sans Docker) : abandonné
- Docker pour les sites vitrines : non utilisé (serverless Firebase à la place)

> Règle : quand un projet est abandonné, extraire les snippets réutilisables vers ce wiki, puis archiver le projet.
