# wiki-vueuse — composables VueUse rares (useDropZone, useObjectUrl, useApi blob, presse-papier)

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

## Presse-papier — copier (`useClipboard`) vs coller image (`useEventListener paste`)

Deux API VueUse différentes, ne pas les mélanger :

| API | Rôle | Sens |
|---|---|---|
| `useClipboard` | **Copier** du texte vers le presse-papier | app → presse-papier |
| `useEventListener(document, 'paste', …)` | **Coller** une image depuis le presse-papier (aperçu + `FormData` POST) | presse-papier → app |

### 1. Copier — composant `ClipBoardBase`

```vue
<template>
  <div :class="{ 'tooltip tooltip-open': copied }" data-tip="Copié !">
    <button type="button" @click="copy(text)" class="btn btn-ghost btn-xs btn-square">
      <i :class="[copied ? 'fas text-primary' : 'fal', 'fa-copy']"></i>
    </button>
  </div>
</template>

<script setup>
import { useClipboard } from '@vueuse/core';

defineOptions({ name: 'ClipBoardBase' });

defineProps({ text: String });

const { copy, copied } = useClipboard({ legacy: true });
</script>
```

Pièges :
- **Jamais de `<button>` imbriqué dans un `<button>`** (HTML invalide) : le tooltip (`<div>`) englobe le bouton.
- `legacy: true` obligatoire : fallback `execCommand('copy')` pour contextes non sécurisés (HTTP local, vieux navigateurs).
- `copied` repasse à false après ~2s (built-in VueUse) : le tooltip `tooltip-open` et l'icône se réinitialisent seuls.
- Icônes : `fal` au repos, `fas` + couleur quand copié — convention [wiki-fontawesome-chat.md](wiki-fontawesome-chat.md).

### 2. Coller une image du presse-papier — `useEventListener(document, 'paste')`

Use case : Cmd+Shift+4 (capture) puis Cmd+V dans la page → aperçu `<img>` + POST `FormData` vers API busboy.

```js
import { useEventListener } from '@vueuse/core';
import { onUnmounted, ref } from 'vue';
import { useApi } from '../../../composables/useApi.js';

const bodyData = ref(null);

// POST différé : immediate false, execute() au submit
const { isFetching: pending, data, execute } = useApi('/prompt/screenshotOCR', { immediate: false }).post(bodyData).text();

const imageUrl = ref(null);

useEventListener(document, 'paste', (event) => {
  const item = event.clipboardData?.items[0];
  if (!item) return;
  if (item.type.startsWith('image/')) {
    const file = item.getAsFile();
    imageUrl.value = URL.createObjectURL(file);
    const formData = new FormData();
    formData.append('clipboard', new File([file], 'screenshot_pasted.png', { type: file.type }));
    bodyData.value = formData;
  }
});

// Nettoyage : révoquer l'objectURL à la destruction du composant
onUnmounted(function () {
  if (imageUrl.value) URL.revokeObjectURL(imageUrl.value);
  imageUrl.value = null;
});
```

Pièges :
- Écouter sur `document` (pas sur le div) : le paste ne cible pas l'élément focus.
- Toujours vérifier `item.type.startsWith('image/')` avant `getAsFile()`.
- `URL.revokeObjectURL` dans `onUnmounted` — sinon fuite mémoire.
- `new File([file], 'screenshot_pasted.png', …)` : renomme le fichier du presse-papier en nom stable côté API.
- Pas besoin de removeEventListener : `useEventListener` le fait à l'unmount.

### Template d'usage (aperçu + submit)

```vue
<div class="min-h-40 border border-dotted w-full rounded-md flex items-center justify-center">
  <input type="text" :value="imageUrl ? 'ok' : ''" style="display: none" required>
  <div v-if="!imageUrl">Faites <strong>Cmd + Shift + 4</strong> ou <strong>Cmd+V</strong> pour coller une capture d'écran ici.</div>
  <img v-else :src="imageUrl" class="w-full rounded-md" alt="clipboard"/>
</div>
```

L'`<input type="text" hidden required>` branché sur `imageUrl` force la validation HTML : le submit est bloqué tant qu'aucune image n'est collée (`e.target.checkValidity()` dans le handler).

Liens : réception du `FormData` côté back → [upload-file-busboy.md](upload-file-busboy.md) ; assemblage complet page upload → [wiki-upload-dragdrop-pdf.md](wiki-upload-dragdrop-pdf.md).

## Complément (fiche upload)

Assemblage complet dans une page (zone drop + label input + aperçu) : [wiki-upload-dragdrop-pdf.md](wiki-upload-dragdrop-pdf.md).

## État

**vivant** — créé oct. 2026 depuis la page PdftoWebp ; section presse-papier fusionnée depuis mode-agent/vueuse-clipboard.md (22/10/2026).
