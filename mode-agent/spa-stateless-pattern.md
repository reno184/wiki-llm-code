# spa-stateless-pattern — État : smart/dumb, pas de store global

## Règles

- **Pas de store global** (Pinia/Vuex : rejetés). L'état vit dans le composant parent smart qui en est propriétaire — pas d'état partagé par singleton, pas de `provide/inject` comme cache global.
- **Un composant = une responsabilité** : le smart porte les données et les appels API, le dumb porte l'affichage et l'interaction pure. Un dumb ne connaît ni `useApi`, ni la structure de l'API.
- **Props drilling assumé** : les données descendent par props du smart vers les dumbs — c'est le mécanisme voulu, pas un anti-pattern à "corriger" avec un store.
- **Callbacks en paramètres** : les remontées d'action passent par des fonctions passées en props au dumb (ex. `@add`), jamais par un accès direct à l'état du parent.
