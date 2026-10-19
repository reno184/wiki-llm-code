# Display MD → HTML

## Contexte

Viewer front pour visualiser le Markdown produit par les workflows de conversion (`use-cases/pdf-to-md`, `correction-syntax-markdown`) : deux formes — brute (source MD) et viewer (rendu HTML), utile pour localiser visuellement les défauts de conversion.

## Stack

Vue 3 (script setup), Tailwind CSS, DaisyUI, VueUse.

## Choix architecture (validé)

**Rendu synchrone `markdown-it` dans un `computed` Vue + composables VueUse.**

- `markdown-it` : parser CommonMark rapide, synchrone, extensible (plugins tableaux/footnotes), standard de facto côté Vue. Alternatives écartées : `marked` (moins extensible), `unified/remark` (API async, overkill ici).
- Le rendu étant synchrone, un simple `computed` suffit — pas de state async ni de watcher.
- VueUse pour l'ergonomie du composant de test : `useLocalStorage` (persistance du MD saisi), `useDebounceFn` (si rendu lourd), `useScroll` (scroll synchronisé raw ↔ viewer en vue split).

## Code

```js
import MarkdownIt from 'markdown-it'
import DOMPurify from 'dompurify'
import { computed } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const md = new MarkdownIt({ html: true, linkify: true, typographer: true })
const raw = useLocalStorage('md-input', '')

// computed = cœur du pattern : re-rendu réactif à chaque édition
const renderedHtml = computed(() => DOMPurify.sanitize(md.render(raw.value)))
```

```html
<!-- viewer -->
<div class="prose" v-html="renderedHtml" />

<!-- raw -->
<pre class="text-xs whitespace-pre-wrap">{{ raw }}</pre>
```

## Briques associées

- **Sanitisation** : `dompurify` obligatoire avant `v-html` (le MD vient de conversions OCR, contenu non maîtrisé).
- **Styling** : plugin `@tailwindcss/typography` (`.prose`) — DaisyUI n'a pas d'équivalent ; les tableaux issus de PDF sont le point dur.
- **Coloration du code brut** (optionnel) : `highlight.js` en plugin markdown-it.

## Notes

- Vue raw indispensable dans les tests : les défauts (niveaux de titres cassés, tableaux mal formés) se voient mieux dans le MD source que dans le rendu.
- Vue split raw ↔ viewer avec `useScroll` pour localiser un défaut sur une zone précise.

État : validé (choix architecture).