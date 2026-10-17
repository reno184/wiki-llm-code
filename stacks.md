# Stacks

## Stack actuelle — sites vitrines

- **Back** : Node.js + Express
- **Templating** : Pug (syntaxe Pug pour les vues)
- **Modules** : CommonJS — `require()`, **pas de `"type": "module"`** (exception à la règle ESM du wiki, voir nodejs-express-env.md)
- **Architecture front** : site **multipage** (pas de SPA)
- **Animations** : JavaScript **View Transitions API** — transitions/animations entre les pages
- **Anti-bot** : reCAPTCHA v3 (voir les fiches reCAPTCHA du wiki)
- **Nom de domaine** : Infomaniak
- **Hosting** : Firebase Hosting
- **Backend serverless** : Firebase Cloud Functions
- **Secrets** : dotenv en local, secrets montés en prod (voir gcp-secrets.md)

## Framework front : Vue.js uniquement

- **Seul framework utilisé : Vue.js** (Vue 3, Composition API).
- Les anciens projets **Angular** et **React** sont **conservés comme exemples et archive**, mais ne sont plus utilisés : ne pas générer de code dessus pour un nouveau projet.

## Technologies abandonnées (ne plus générer de code dessus)

- **TypeScript** : abandonné — utiliser JavaScript.
- **Vuetify** : abandonné comme librairie UI.
- Angular + Firebase (sans Docker) : projet archivé, exemple uniquement.
- React : projets archivés, exemples uniquement.
- Docker pour les sites vitrines : non utilisé (serverless Firebase à la place).
