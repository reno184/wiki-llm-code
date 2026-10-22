# VueUse : clipboard (copier dans le presse-papier)

Composant `ClipBoardBase` — bouton copier réutilisable (btn-square daisyUI + icône FontAwesome + tooltip "Copié !").

## Composant complet

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

## Pièges d'exécution

- **Jamais de `<button>` imbriqué dans un `<button>`** (HTML invalide, comportement imprévisible) : le tooltip (`<div>`) englobe le bouton, il ne s'imbrique pas.
- `legacy: true` obligatoire : fallback `execCommand('copy')` pour contextes non sécurisés (HTTP local, vieux navigateurs).
- `copied` repasse à false après ~2s (built-in VueUse) : le tooltip `tooltip-open` et l'icône `fas text-primary` se réinitialisent seuls.
- Icônes : `fal` au repos, `fas` + couleur quand copié — même convention que [wiki-fontawesome-agent.md](wiki-fontawesome-agent.md).

## Liens

- Styling btn/tooltip : [daisyui-component.md](daisyui-component.md).
- Icônes : [wiki-fontawesome-agent.md](wiki-fontawesome-agent.md).
