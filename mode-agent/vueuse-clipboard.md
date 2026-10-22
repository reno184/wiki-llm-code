# VueUse : presse-papier — copier (useClipboard) vs coller image (useEventListener paste)

## ⚠️ Anti-confusion (à relire dans 6 mois)

Deux API VueUse différentes, ne pas les mélanger :

| API | Rôle | Sens |
|---|---|---|
| `useClipboard` | **Copier** du texte vers le presse-papier | app → presse-papier |
| `useEventListener(document, 'paste', …)` | **Coller** une image depuis le presse-papier dans le composant, l'afficher et la passer en `FormData` pour un POST | presse-papier → app |

- `useEventListener(document, 'paste')` = **coller/afficher l'image** dans le composant + construire le `FormData` pour le POST (use case : coller une capture d'écran → POST vers API busboy).
- `useClipboard` = **uniquement copier/coller du texte** (bouton copier). Rien à voir avec l'upload d'image.

## 1. Copier — composant `ClipBoardBase`

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
- Icônes : `fal` au repos, `fas` + couleur quand copié — convention [wiki-fontawesome-agent.md](wiki-fontawesome-agent.md).

## 2. Coller une image du presse-papier — `useEventListener(document, 'paste')`

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

## Template d'usage (aperçu + submit)

```vue
<div class="min-h-40 border border-dotted w-full rounded-md flex items-center justify-center">
  <input type="text" :value="imageUrl ? 'ok' : ''" style="display: none" required>
  <div v-if="!imageUrl">Faites <strong>Cmd + Shift + 4</strong> ou <strong>Cmd+V</strong> pour coller une capture d'écran ici.</div>
  <img v-else :src="imageUrl" class="w-full rounded-md" alt="clipboard"/>
</div>
```

L'`<input type="text" hidden required>` branché sur `imageUrl` force la validation HTML : le submit est bloqué tant qu'aucune image n'est collée (`e.target.checkValidity()` dans le handler).

## Liens

- Réception du `FormData` côté back : [../backend/upload-file-busboy.md](../backend/upload-file-busboy.md).
- POST avec `useApi` (immediate false, execute) : [daisyui-component.md](daisyui-component.md).
- Styling btn/tooltip : [daisyui-component.md](daisyui-component.md) — icônes : [wiki-fontawesome-agent.md](wiki-fontawesome-agent.md).
