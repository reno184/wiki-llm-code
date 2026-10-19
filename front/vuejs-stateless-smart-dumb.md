# Architecture composants — stateless, smart/dumb, props + callbacks (Vue 3)

Stack: Vue 3 Composition API, @vueuse/core (useApi), aucune lib de state (pas de Pinia).
Usage: **règle d'architecture front pour tout agent IA** — les composants sont codés en stateless, sans store global. Découpage smart/dumb à responsabilité unique, données portées par les props vers le bas, actions remontées par des callbacks passées en paramètre. Fiche transversale front, à charger avec [vuejs-useapi.md](../vuejs-useapi.md) et [javascript/form-validation.md](javascript/form-validation.md).

## Règles pour les agents

- **Pas de store global** (Pinia/Vuex : rejetés). L'état vit dans le composant parent smart qui en est propriétaire — pas d'état partagé par singleton, pas d'`provide/inject` comme cache global.
- **Un composant = une responsabilité** : le smart porte les données et les appels API, le dumb porte l'affichage et l'interaction pure. Un dumb ne connaît ni `useApi`, ni la structure de l'API.
- **Props drilling assumé** : les données descendent par props du smart vers les dumbs — c'est le mécanisme voulu, pas un anti-pattern à "corriger" avec un store.
- **Callbacks en paramètres** : les remontées d'action passent par des fonctions passées en props au dumb (ex. `@add`), jamais par un accès direct à l'état du parent.

## Pattern type — bouton d'ajout + liste (3 composants)

Le parent smart porte la liste + **2 `useApi`** : le GET (immediate `true`, chargé au montage du parent) et le POST (immediate `false`, déclenché par l'ajout). Après un POST réussi, le smart rappelle le GET pour rafraîchir la liste.

```js
import { ref } from 'vue'

const list = ref([])
const bodyData = ref(null)

const { data, execute: getList } = useApi('/api/items').get().json()

const { isFetching: pending, execute: postItem } = useApi('/api/items', { immediate: false }).post(bodyData).json()
```

- `useApi` GET : **immediate `true` par défaut** — la liste est appelée quand le composant parent est monté, `data` alimente la liste.
- `useApi` POST : `{ immediate: false }` — ne part qu'au submit (voir [javascript/form-validation.md](javascript/form-validation.md)).
- `execute: getList` : le GET est nommé pour être **rappelé explicitement** — le rafraîchissement de la liste après un POST réussi est un appel délibéré, pas un effet magique.
- `list` en `ref` réactif dans le smart : les dumbs le reçoivent en props et se re-rendent seuls au refresh.

Structure des 3 composants :

- **Parent smart** : possède `list`, les 2 `useApi`, `getList()` après succès du POST. Il ne rend que l'assemblage + le dispatch des callbacks.
- **Dumb bouton d'ajout** : formulaire/champ + bouton, émet l'ajout via callback (`@add` ou props function) — ne sait pas qu'un POST existe.
- **Dumb liste** : rend `list` reçu en props, key par item — ne sait pas d'où viennent les données.

## Notes

- Fiche **en construction** : les détails seront complétés par la suite (découpage exact, cas du refresh après POST, exemples de dumbs types).
- Le refresh par re-GET après POST (plutôt qu'une mise à jour locale optimiste de `list`) est le choix par défaut de simplicité — à confirmer/compléter.
- Liens : [vuejs-useapi.md](../vuejs-useapi.md) (wrapper, erreurs, conventions isFetching), [javascript/form-validation.md](javascript/form-validation.md) (submit + immediate false), [daisyui-components.md](../daisyui-components.md).

## État

**en construction** — règles posées pour les agents (stateless, smart/dumb, props + callbacks, 2 useApi parent) ; consolidation avec les détails à venir.
