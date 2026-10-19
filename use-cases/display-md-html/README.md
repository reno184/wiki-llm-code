# Display Markdown into Vue component

## Contexte

Viewer front pour visualiser le Markdown produit par les workflows de conversion (`use-cases/pdf-to-md`, `correction-syntax-markdown`) : deux formes — brute (source MD) et viewer (rendu HTML), utile pour localiser visuellement les défauts de conversion.

Déploiement : apps internes en Docker dans des cabinets d'avocats (localhost / réseau privé, pas d'exposition internet).

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

const props = defineProps<{ raw: string }>()

// html: false = décision sécurité : tout HTML brut dans le MD est rendu
// en texte échappé, jamais interprété. Pas besoin de DOMPurify.
const md = new MarkdownIt({ html: false, linkify: true, typographer: true })

// computed = cœur du pattern : re-rendu réactif à chaque édition de la prop
const renderedHtml = computed(() => md.render(props.raw))
</script>

<template>
  <!-- viewer : rendu HTML, HTML brut du MD échappé par markdown-it -->
  <div class="prose" v-html="renderedHtml" />

  <!-- raw : source MD brute -->
  <pre class="text-xs whitespace-pre-wrap">{{ raw }}</pre>
</template>
```

## Sécurité — décision : `html: false`, DOMPurify écarté

**Le risque `v-html` vient du HTML brut éventuellement présent dans le Markdown source** (PDF externes non maîtrisés : documents clients, pièces adverses — un PDF piégé peut contenir du HTML/JS qui survit à la conversion OCR).

Décision retenue : **`new MarkdownIt({ html: false })`**.

- Tout HTML brut du Markdown (`<script>`, `<img onerror>`…) est rendu **en texte échappé**, jamais interprété par le navigateur. La surface XSS dans `v-html` est supprimée à la source, sans librairie supplémentaire.
- Déploiement interne (Docker, localhost, cabinets d'avocats) : on accepte le trade-off.
- **Trade-off assumé** : les fragments HTML utiles issus des PDF ne sont pas rendus (visibles en texte brut dans le viewer — ce qui est d'ailleurs un bon signal visuel de défaut de conversion).
- **Angle mort restant** : les liens `javascript:` générés via `linkify` ou Markdown explicite ne sont PAS neutralisés par `html: false`. Ne pas brancher de plugin `linkify` sur des sources non maîtrisées sans filtre, ou filtrer les schémas d'URL si le besoin apparaît (règle par défaut du validateur de liens de markdown-it à configurer).

Réévaluer si : exposition internet, ajout de `html: true` (fragment HTML nécessaire), ou contenu provenant d'utilisateurs externes.

## Styling & coloration (affichage uniquement)

- **Styling** : plugin `@tailwindcss/typography` (`.prose`) — DaisyUI n'a pas d'équivalent ; les tableaux issus de PDF sont le point dur.
- **Coloration du code** (optionnel) : `highlight.js` en plugin markdown-it.

## VueUse — uniquement ce qui sert à l'affichage

- `useScroll` : scroll synchronisé raw ↔ viewer en vue split (localiser un défaut sur une zone précise).

Rien d'autre de VueUse n'entre dans ce workflow : la persistance (`useLocalStorage`) et le debounce relèvent du composant de saisie amont, pas de l'affichage.

## Notes

- Vue raw indispensable dans les tests : les défauts (niveaux de titres cassés, tableaux mal formés) se voient mieux dans le MD source que dans le rendu.

État : validé (choix architecture + décision sécurité `html: false`).