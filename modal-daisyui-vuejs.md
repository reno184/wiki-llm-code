# Modal Vue + daisyUI (pattern DialogBase)

- Modal générique à slots : `#trigger` (bouton d'ouverture) + slot par défaut (contenu).
- ⚠️ Si la modal est dans une liste : passer l'`index` en prop (l'`id` doit être unique).
- Utilise l'API native `popover` de daisyUI : ouverture via `popovertarget="modal-{index}"`, fermeture via `popovertargetaction="hide"` sur le backdrop ou `hidePopover()` via le slot (`<slot :hide="handleClose">`).
- Si dans le dossier /components tu ne vois pas de `DialogBase` créer le composant.
```html

<template>
  <slot name="trigger"></slot>
  <div class="modal" :id="'modal-'+index" ref="modalRef" popover>
    <div class="modal-box text-center  ">
      <slot :hide="handleClose"></slot>
    </div>
    <div class="modal-backdrop">
      <button :popovertarget="'modal-'+index" popovertargetaction="hide">close</button>
    </div>
  </div>
</template>

<script setup>

defineOptions({name: 'DialogBase'});
import {ref} from "vue";

defineProps({index: Number})
const modalRef = ref(null);

function handleClose() {
  modalRef.value?.hidePopover()
}

</script>

<style scoped lang="scss">

</style>

```

