# spa-arborescence-pattern — Rangement et nommage : composants vs views

## Règle

- **`src/views/` = uniquement les pages appelées par `router/index.js`**, suffixées `Page` : `ConvertWebpPage.vue`, `OcrExtractPage.vue`.
- **Toutes les vues qui ne sont pas dans `router/index.js`** : vont dans le dossier `src/components/`.
