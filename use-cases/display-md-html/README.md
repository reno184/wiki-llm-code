# Display Markdown into Vue component

## Contexte

Viewer front pour visualiser le Markdown produit par les workflows de conversion (`use-cases/pdf-to-md`, `correction-syntax-markdown`) : deux formes — brute (source MD) et viewer (rendu HTML), utile pour localiser visuellement les défauts de conversion.

## Stack

Vue 3 (script setup), Tailwind CSS, DaisyUI.

## Choix architecture (validé)

**Rendu synchrone `markdown-it` dans un `computed` du composant Vue.**

- `markdown-it` : parser CommonMark rapide, synchrone, extensible (plugins tableaux/footnotes), standard de facto côté Vue. Alternatives écartées : `marked` (moins extensible), `unified/remark` (API async, overkill ici).
- Le rendu étant synchrone, un simple `computed` suffit — pas de state async ni de watcher : le composant re-rend à chaque changement de la source MD.

## Code — composant `MarkdownViewer.vue`

```vue
<script setup>
import { computed } from 'vue'
import MarkdownIt from 'markdown-it'
import DOMPurify from 'dompurify'

const props = defineProps<{ raw: string }>()

const md = new MarkdownIt({ html: true, linkify: true, typographer: true })

// computed = cœur du pattern : re-rendu réactif à chaque édition de la prop
const renderedHtml = computed(() => DOMPurify.sanitize(md.render(props.raw)))
</script>

<template>
  <!-- viewer : rendu HTML sanitisé -->
  <div class="prose" v-html="renderedHtml" />

  <!-- raw : source MD brute -->
  <pre class="text-xs whitespace-pre-wrap">{{ raw }}</pre>
</template>
```

## Briques associées (affichage uniquement)

- **Sanitisation** : `dompurify` obligatoire avant `v-html` (le MD vient de conversions OCR, contenu non maîtrisé).
- **Styling** : plugin `@tailwindcss/typography` (`.prose`) — DaisyUI n'a pas d'équivalent ; les tableaux issus de PDF sont le point dur.
- **Coloration du code** (optionnel) : `highlight.js` en plugin markdown-it.

## VueUse — uniquement ce qui sert à l'affichage

- `useScroll` : scroll synchronisé raw ↔ viewer en vue split (localiser un défaut sur une zone précise).

Rien d'autre de VueUse n'entre dans ce workflow : la persistance (`useLocalStorage`) et le debounce relèvent du composant de saisie amont, pas de l'affichage.

## Notes

- Vue raw indispensable dans les tests : les défauts (niveaux de titres cassés, tableaux mal formés) se voient mieux dans le MD source que dans le rendu.

État : validé (choix architecture).