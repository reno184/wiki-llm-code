---
name: display-md
layer: front
stack: Vue 3, Tailwind CSS, DaisyUI, VueUse, markdown-it, DOMPurify
status: choix-validé
---

# Use case display-md

Visualisation de Markdown (résultat des conversions PDF → MD) dans une app Vue 3 — deux formes : **raw** (source) et **viewer** (rendu HTML).

## Choix d'architecture

**markdown-it + `computed` Vue + VueUse.**

- Parser : `markdown-it` (CommonMark, plugins tableaux/footnotes, rendu synchrone — s'intègre naturellement dans un `computed`).
- Rendu réactif : `computed(() => md.render(props.rawMarkdown))` — pas d'état intermédiaire, recalcul minimal.
- VueUse : `useLocalStorage` (persistance du MD / du mode), `useDebounceFn` (édition → rendu), `useScroll` (scroll synchronisé raw/viewer en split).
- Sanitisation : `DOMPurify.sanitize()` obligatoire avant `v-html` (le MD vient d'OCR/conversion, source non fiable).
- Styling viewer : `@tailwindcss/typography` (`prose`) — DaisyUI n'a pas d'équivalent.
- UI : toggle DaisyUI (tabs ou swap) pour les modes raw / viewer / split.

```js
import MarkdownIt from 'markdown-it'
import DOMPurify from 'dompurify'
import { computed } from 'vue'

const md = new MarkdownIt({ html: true, linkify: true, typographer: true })
const sanitizedHtml = computed(() => DOMPurify.sanitize(md.render(props.raw)))
```

```html
<div class="prose" v-html="sanitizedHtml" />
```

## Règles

- Toujours garder la vue **raw** accessible : les défauts de conversion (niveaux de titres, tableaux mal formés — cf. `correction-syntax-markdown`) se voient mieux dans la source que dans le rendu HTML.
- Alternatives écartées : `marked` (moins extensible), `unified/remark` (async, verbeux — overkill pour ce besoin).
- Édition du MD : CodeMirror 6 plutôt qu'un `<textarea>` le jour où le besoin apparaît.

## Position dans la chaîne

`pdf-to-md` → `correction-syntax-markdown` → **`display-md`** (visualisation/validation humaine) → `anonymisation`
