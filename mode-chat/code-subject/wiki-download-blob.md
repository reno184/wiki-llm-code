# Download de fichier — front (Blob → navigateur, saveAs natif)

Stack: Vue 3 / JS natif — pattern `createObjectURL` + `<a download>` pour déclencher un téléchargement navigateur depuis un Blob reçu d'une API (pas de fichier statique, pas de lib type file-saver).

Récupération du Blob côté Vue : composable VueUse `useApi` (voir [wiki-vueuse.md](wiki-vueuse.md)) avec `.blob()` qui renvoie un `Blob` au lieu du JSON parsé :

```js
const { data: uploadBlob, execute: encryptUpload } = useApi('/api/python/encrypt/upload', { immediate: false })
  .post(bodyUpload)
  .blob()
```

⚠️ `immediate: false` + `execute()` déclenché au clic (pas de requête au montage). Le `data` est la **réponse brute** (Blob) quand l'API renvoie `Content-Type: application/octet-stream` ou `application/pdf` avec `Content-Disposition: attachment`.

## Pattern minimal de téléchargement

```js
if (blob) {
  const url = window.URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = \`\${currentId.value || Date.now()}-anonymised.pdf\`
  a.click()
  window.URL.revokeObjectURL(url)
}
```

## Checklist « ne rien oublier »

- **`a.download`** : nom de fichier imposé côté client. Sans cet attribut, le navigateur **navigue** vers le blob (affichage/onglet) au lieu de télécharger.
- **Nom dynamique** : préfixe métier (ex. id courant) avec fallback `Date.now()` pour garantir l'unicité.
- **`createObjectURL` puis `revokeObjectURL`** : le blob reste en mémoire tant que l'URL n'est pas révoquée — toujours révoquer après le `click()` (synchrone ici, le download est déjà programmé).
- L'`<a>` n'a **pas besoin d'être dans le DOM** : `document.createElement` + `click()` suffit.
- Le `<a>` est jetable, un par téléchargement (pas de réutilisation).

## Pièges

- `useApi(...).blob()` **sans** `.blob()` → le body binaire part dans un `JSON.parse` et casse. Le mode d'appel (`.json()` par défaut / `.blob()`) fait partie du contrat d'API.
- Ne pas confondre avec `useObjectUrl` (VueUse, aperçu affichable dans le DOM avec révocation auto) — ici on veut un **téléchargement**, pas un affichage ; la révocation est manuelle et immédiate.
- Si l'API renvoie le fichier en **multipart/plusieurs fichiers**, ce pattern ne suffit pas → côté back voir [upload-file-busboy.md](upload-file-busboy.md) (mêmes idées inversées).

## Liens croisés

- [wiki-vueuse.md](wiki-vueuse.md) — `useApi` `immediate: false` + `execute`, `useObjectUrl` (aperçu ≠ download)
- [wiki-upload-dragdrop-pdf.md](wiki-upload-dragdrop-pdf.md) — le sens inverse (upload PDF drag & drop), même pipeline
- [wiki-minio-pdf.md](wiki-minio-pdf.md) — stockage/serve des fichiers côté back
