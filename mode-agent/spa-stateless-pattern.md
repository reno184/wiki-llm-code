# spa-stateless-pattern — État : smart/dumb, pas de store global

## Règles

- **Pas de store global** (Pinia/Vuex : rejetés). L'état vit dans le composant parent smart qui en est propriétaire — pas d'état partagé par singleton, pas de `provide/inject` comme cache global.
- **Le parent (smart) ne fait que le GET de la liste** : il possède les données de liste et l'appel API `get`. Les remontées d'action ne remontent pas au parent.
- **Chaque enfant fait son propre appel d'écriture** : un formulaire fait lui-même son POST, un bouton delete fait lui-même son DELETE. Le call vit au plus près de l'UI qui le déclenche — code moins verbeux, pas de callbacks en props pour les actions.
- **Pending sur le bon bouton** : `isFetching` de `useApi` est porté par le composant qui fait le call — le spinner de chargement s'affiche sur le bouton de la ligne concernée (cf. [daisyui-component.md](daisyui-component.md) btn loading), pas un état global de page.
- Après une écriture réussie (post/delete), rafraîchir la liste du parent.
- **Un composant = une responsabilité** : le parent porte les données de liste et le GET ; les enfants portent leur propre action d'écriture et l'affichage. Un enfant ne connaît pas la structure complète de l'API au-delà de sa propre route.
- **Props drilling assumé** : les données de liste descendent par props du parent vers les enfants — c'est le mécanisme voulu, pas un anti-pattern à "corriger" avec un store.
- ** callback function** : ne pas utiliser de `defineEmits(['onRefresh'])`, mais `defineProps` avec un callBack function en proprité `onRefrech: Function`
