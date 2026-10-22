# VueUse — liste des composables utilisés

Liste de vérification rapide pour l'agent : AVANT d'implémenter un utilitaire front (clipboard, storage, events, debounce…), vérifier ici si un composable VueUse existe déjà. 1 ligne par composable. Fiche détaillée si nécessaire (cf. colonne Fiche).

**Exception `useApi`** : composable custom (pas VueUse) — NE PAS le réimplémenter, il existe déjà dans `../composables/useApi.js`.

| Composable | Cas d'usage (~3 mots) | Fiche |
|---|---|---|
| `useClipboard` | copier texte bouton | [vueuse-clipboard.md](vueuse-clipboard.md) |
| `useEventListener` | écouter event auto-unmount | [vueuse-clipboard.md](vueuse-clipboard.md) |
| `useObjectUrl` | aperçu image locale auto-revoke | — |
| `useDropZone` | drop fichiers zone upload | — |

Règle : à chaque nouveau composable adopté, ajouter une ligne ici (même trivial) — sinon l'agent le réimplémente à la main.
