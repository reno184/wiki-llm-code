# wiki-upload-dragdrop-pdf — upload drag & drop PDF (VueUse + useApi blob)

Usage: page d'upload avec zone de glisser-déposer PDF, envoi FormData vers l'API, affichage du résultat binaire (Blob → image). Snippet rare (~1×/2 mois), copié/collé manuel.

## Dépendances

- `@vueuse/core` : `useDropZone`, `useObjectUrl`
- composable `useApi` (front/vuejs-useapi.md)

## Pattern minimal

```vue
<template>
  <div ref="dropZoneRef" :class="{ 'bg-base-200': isOverDropZone }"
       class="rounded-lg border-2 border-dashed border-base-300 p-8 text-center italic">
    <span>Glissez-déposez vos éléments ici</span>
  </div>

  <label class="btn cursor-pointer">
    <span>Sélectionner un fichier</span>
    <input type="file" class="hidden" accept=".pdf,application/pdf" @change="e => onChange(e.target.files)"/>
  </label>

  <div v-if="imageObjectUrl">
    <img :src="imageObjectUrl" alt="Aperçu" />
  </div>
</template>

<script setup>
import {useDropZone, useObjectUrl} from '@vueuse/core'
import {ref} from 'vue'
import {useApi} from '../composables/useApi.js'

defineOptions({name: 'UploadPage'})

const dropZoneRef = ref(null)

const {isOverDropZone} = useDropZone(dropZoneRef, {
  onDrop: onChange,
  multiple: false,
  preventDefaultForUnhandled: false,
})

const bodyData = ref(null)

const {execute, data: blobData} = useApi('/route', {immediate: false}).post(bodyData).blob()

const imageObjectUrl = useObjectUrl(blobData)

async function onChange(files) {
  if (!files || files.length === 0) return

  const file = files[0]

  const formData = new FormData()
  formData.append('file', file)

  bodyData.value = formData

  await execute()
}
</script>
```

## Points clés

- `useDropZone(dropZoneRef, {onDrop: onChange})` : le ref est sur le div de la zone ; le `@change` du label input pointe la même `onChange` — une seule fonction pour les deux entrées.
- `useApi(...).post(bodyData).blob()` avec `immediate: false` + `await execute()` : l'appel part quand le FormData est prêt.
- `useObjectUrl(blobData)` : transforme le Blob réponse en URL locale affichable (`blob:http://...`), révoquée automatiquement.
- `accept=".pdf,application/pdf"` : filtre la sélection ; contrôle du type dans `onChange` si besoin.
- Styles daisyUI à adapter au projet (l'original : hero `bg-olive-100`, bordure dashed `border-base-300`).

## État

**vivant** — réduit depuis le snippet AnonymisationPage (oct. 2026) pour stockage mode-chat (usage rare, pas de coût tokens agent).
