# Pattern composants

- Pas de store global : état dans le parent (smart). Enfants dumb/stateless.
- Parent : GET liste + détient les données, descend par props (props drilling assumé).
- Enfant : fait ses propres écritures (POST/DELETE) au plus près de l'UI qui les déclenche. Pas de callbacks d'action en props.
- `isFetching` de `useApi` porte le pending dans le composant qui appelle → spinner sur le bouton de la ligne concernée (cf. daisyui-component.md `btn loading`).
- Après écriture OK → rafraîchir la liste parent via callback.
- Callbacks uniquement en props (pas de `defineEmits`), préfixés `on` : `onRefresh: Function` (refresh liste parent), `onClose` (fermer modal/popover).