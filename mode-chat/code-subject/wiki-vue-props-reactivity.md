# wiki-vue-props-reactivity

Destructuration des props & réactivité dans **Vue 3.5+** — pièges et réflexes à ne pas oublier en copié/collé manuel.

## 1. Props destructurées (Vue 3.5+)

Depuis **Vue 3.5**, la destructuration via `defineProps` conserve la réactivité (*Reactive Props Destructure* natif).

```javascript
const { url, pdfList, onChange } = defineProps({
  url: String,
  pdfList: Array,
  onChange: Function
})
```

- **Ce n'est pas une `ref`** : les propriétés destructurées se manipulent comme des variables JS classiques.
- **Pas de `.value`** : accès direct (`url`, `pdfList.length`…).
- **Réactivité préservée** : si le parent modifie la prop, la variable destructurée se met à jour dans l'enfant.

## 2. Watcher sur une prop destructurée

Passer une **fonction getter**, pas la variable seule :

```javascript
import { watch } from 'vue'

watch(() => url, (newUrl) => {
  console.log('L\'URL a changé :', newUrl)
})
```

## 3. Lecture seule (Unidirectional Data Flow)

Les props reçues du parent sont **toujours en lecture seule** dans l'enfant :

- Ne jamais muter directement un objet/tableau reçu en prop (éviter `pdfList.push(...)` ou `pdfList[0].name = '...'`).
- La mutation directe crée un *two-way binding* involontaire et rend l'état imprévisible.

## 4. Notifier le parent (callback / emit)

L'enfant passe une **nouvelle référence** (objet/tableau) via callback ou `emit` ; c'est le parent qui mute sa propre référence.

```javascript
// Dans l'enfant
function handleSelect(pdfId) {
  const pdfItem = pdfList.find(x => x.pdfId === pdfId)
  onChange({ ...pdfItem })  // nouvelle référence au parent
}
```

```javascript
// Dans le parent
function handleSelectChange(newPdf) {
  selectedPdf.value = newPdf  // le parent met à jour sa référence réactive
}
```

## Récapitulatif

| Concept | Règle |
| :--- | :--- |
| Accès à la prop (3.5+) | Variables directes, **pas de `.value`** |
| Type | Pas des `ref`, mais **réactives** |
| Observer (`watch`) | Getter : `watch(() => prop, callback)` |
| Modification | **Lecture seule** — callback/emit vers le parent avec une nouvelle référence |

## Liens croisés

- [wiki-vueuse.md](wiki-vueuse.md) — composables VueUse (useDropZone, useObjectUrl, useApi)
- [wiki-upload-dragdrop-pdf.md](wiki-upload-dragdrop-pdf.md) — front Vue : upload drag & drop PDF
