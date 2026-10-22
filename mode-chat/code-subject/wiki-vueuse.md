# wiki-vueuse — composables VueUse rares (useDropZone, useObjectUrl, useApi blob)

Usage : référence des composables `@vueuse/core` utilisés rarement — signatures, pièges, options. Pattern complet d'une page d'upload : [wiki-upload-dragdrop-pdf.md](wiki-upload-dragdrop-pdf.md) (la fiche ici ne répète pas le code de la page).

## useDropZone

```js
import {useDropZone} from '@vueuse/core'

const dropZoneRef = ref(null)

const {isOverDropZone} = useDropZone(dropZoneRef, {
  onDrop: onChange,
  multiple: false,
  preventDefaultForUnhandled: false,
})
```

- Le `ref` est sur le `div` de la zone (pas sur l'input). `isOverDropZone` = bool réactif pour le style hover (`:class="{ 'bg-base-200': isOverDropZone }"`).
- `onDrop` reçoit un `FileList` : même signature que `e.target.files` → brancher la même fonction `onChange` que le `@change` du label input caché. Une seule fonction pour les deux entrées.
- `multiple: false` → toujours un tableau `files`, vérifier `files?.length` avant `files[0]`.
- Le filtrage de type (`accept=".pdf,application/pdf"` sur l'input) ne s'applique PAS au drag & drop : contrôler `file.type` / extension dans `onChange`.

## useObjectUrl

```js
const imageObjectUrl = useObjectUrl(blobData)
```

- Prend un `Blob` (réponse binaire API) et renvoie une URL locale affichable (`blob:http://...`) pour `<img :src="...">`.
- Révocation automatique quand la source change ou au démontage — ne pas gérer `URL.createObjectURL`/`revokeObjectURL` à la main.
- `v-if="imageObjectUrl"` avant d'afficher l'aperçu (null tant que pas de réponse).

## useApi en mode blob (aperçu d'une réponse binaire)

```js
const bodyData = ref(null)

const {execute, data: blobData} = useApi('/api/route', {immediate: false}).post(bodyData).blob()
```

- `immediate: false` + `bodyData` en ref : remplir `bodyData.value = formData` puis `await execute()` — l'appel part quand le FormData est prêt.
- `.blob()` → `data` est un `Blob`, à brancher sur `useObjectUrl` (pas de `JSON.parse`).
- Signature d'origine : composable `useApi` (voir front/vuejs-useapi.md).

## Complément (fiche upload)

Assemblage complet dans une page (zone drop + label input + aperçu) : [wiki-upload-dragdrop-pdf.md](wiki-upload-dragdrop-pdf.md).

## État

**vivant** — créé oct. 2026 depuis la page PdftoWebp ; composables rares → mode-chat, copié/collé manuel.
