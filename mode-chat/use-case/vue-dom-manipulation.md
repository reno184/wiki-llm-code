# Manipulation DOM vanilla JS dans Vue (querySelector, click, nextTick)

Use-case : parent → enfant → petit-enfant, `v-if` piloté par `isVisible = ref(true)`, manipulation DOM impérative obligatoire (querySelector…).

## Règle 1 — nextTick avant toute lecture du DOM

Vue patche le DOM de façon asynchrone (batch). Après `isVisible.value = true`, le nœud n'existe pas encore.

```js
import { nextTick } from 'vue'
isVisible.value = true
await nextTick()
document.querySelector('.enfant') // ✅ sinon null → crash
```

## Règle 2 — propagation par défaut d'un click (bubbling)

Un clic sur le petit-enfant remonte la chaîne DOM : **3 handlers se déclenchent, du plus profond au plus superficiel** — petit-enfant, puis enfant, puis parent.

Couper à la source :

```html
<div @click.stop="onPetitEnfantClick">…</div>
``

Sans `.stop`, un clic sur le petit-enfant déclenche accidentellement le `isVisible.value = false` du parent.

## Règle 3 — le piège combiné v-if + bubbling + querySelector

1. Clic petit-enfant → bulle jusqu'au parent.
2. Handler parent : `isVisible.value = false` → le `v-if` démonte l'enfant ET tout son contenu.
3. Un handler/callback exécuté ensuite fait un querySelector sur la div enfant → **null / TypeError**.

Parades :
- `@click.stop` sur petit-enfant (et enfant si besoin).
- Toujours vérifier avant de manipuler :

```js
const el = document.querySelector('.enfant')
if (el) { /* … */ }   // ou el?.doSomething()
```

- Vérifier `isVisible.value` dans le handler parent avant toute manipulation.

## Pièges complémentaires

- **v-if vs v-show** : `v-show` garde le nœud en mémoire (display:none) — pas de crash, pas de perte d'état interne.
- **Template refs > querySelector** : `<div ref="monEl">` + `const monEl = ref(null)` ; le ref passe à null automatiquement au démontage, testable `if (monEl.value)`.
- **emit ne bulle pas** : un événement custom Vue n'atteint que le parent qui écoute `@mon-event` ; seul un `@click` natif sur la racine d'un composant bulle.
- **Handlers déjà en file** : après `isVisible.value = false`, les autres handlers du même clic s'exécutent quand même — seule `.stop` (ou stopImmediatePropagation) coupe.
- **addEventListener manuel** : le retirer dans `onBeforeUnmount` (fuite mémoire / handler fantôme).

## Liens croisés

- [wiki-vue-props-reactivity.md](../code-subject/wiki-vue-props-reactivity.md) — enfant notifie le parent par emit avec nouvelle référence
- [wiki-vuejs-reset.md](../code-subject/wiki-vuejs-reset.md) — shell projet Vue
- [wiki-vueuse.md](../code-subject/wiki-vueuse.md) — composables VueUse (écouteurs/nettoyage auto)
