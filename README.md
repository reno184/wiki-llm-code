# wiki-llm-code

Bibliothèque personnelle de snippets et patterns de code réutilisables, au format OKF (fichiers markdown autonomes).
Objectif : orienter la génération de code par LLM (Vibe/Mistral) sur des patterns déjà validés.

## Règles

- **Une seule version vivante par besoin** (ex. un seul pattern de pagination). Les anciennes versions partent dans `_archive/` avec la mention "remplacé par X".
- Chaque fiche est autonome : stack, usage, code complet, notes, état.
- `INDEX.md` est le point d'entrée : c'est le seul fichier à donner à un LLM en priorité.
- Range par couche (front/backend/infra/patterns), jamais par projet.

## Structure

- `INDEX.md` — index de toutes les fiches (stack, usage, état vivant/obsolete)
- `front/` — snippets front (Vue 3, Tailwind, DaisyUI, VueUse)
- `backend/` — snippets back / API (Node/Express)
- `infra/` — Docker, déploiement, CI
- `patterns/` — patterns de développement (SOLID, etc.)
- `stacks.md` — stack de référence + stacks abandonnées
- `_archive/` — snippets remplacés (historique, jamais consultés)
