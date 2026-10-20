# INDEX — mode-agent

Fiches pour agents (Vue/Vite + daisyUI front, Express back). Jamais chargées automatiquement — consultation manuelle via MCP GitHub.

| Fiche | Usage | État |
|---|---|---|
| [code-style.md](code-style.md) | Style code : toujours des fonctions nommées | vivant |
| [spa-arborescence-pattern.md](spa-arborescence-pattern.md) | Rangement SPA : `src/views/` = pages du router uniquement (suffixe `Page`), le reste → `src/components/` | vivant |
| [spa-stateless-pattern.md](spa-stateless-pattern.md) | État SPA : pas de store global, parent = GET liste uniquement, chaque enfant fait son propre POST/DELETE, pending sur le bouton de la ligne | vivant |
| [modal-vuejs.md](modal-vuejs.md) | Modal générique `DialogBase` (daisyUI popover, slots `#trigger` + contenu, `index` en prop si liste) | vivant |
| [daisyui-component.md](daisyui-component.md) | Composants DaisyUI : btn loading (spinner sur `isFetching` de `useApi`), loader infinity-xl pour invoke LangChain (traitement long) | vivant |
| [wiki-fontawesome-agent.md](wiki-fontawesome-agent.md) | Icônes front : balise `<i>`, solid par défaut (`fa`), light `fal`, icônes usuelles | vivant |
| [vuejs-reset.md](vuejs-reset.md) | Reset projet Vue : page Erreur 500 — chercher/créer `Error500Page.vue` dans `/views` (stack en dev, message simple en prod) | vivant |
| [api-express.md](api-express.md) | Stack backend Node/Express : logique dans la route, try/catch + `next(err)`, `AppError` (2 exceptions 400), vérifier `express.json()` | vivant |

## Liens entre fiches

- Vue SPA complet : [spa-arborescence-pattern.md](spa-arborescence-pattern.md) (rangement) + [spa-stateless-pattern.md](spa-stateless-pattern.md) (état) + [modal-vuejs.md](modal-vuejs.md) (modals) + [daisyui-component.md](daisyui-component.md) (btn/loading) + [wiki-fontawesome-agent.md](wiki-fontawesome-agent.md) (icônes) + [vuejs-reset.md](vuejs-reset.md) (erreur 500).
- Pattern liste : [spa-stateless-pattern.md](spa-stateless-pattern.md) (parent GET, enfant POST/DELETE) → [daisyui-component.md](daisyui-component.md) (pending sur le bouton de la ligne) → [modal-vuejs.md](modal-vuejs.md) (modal d'édition/suppression dans une liste : `index` en prop).
- Style transversal : [code-style.md](code-style.md) (fonctions nommées) s'applique à toutes les fiches front + [api-express.md](api-express.md) côté back.
