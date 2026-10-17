# wiki-llm-code

Bibliothèque personnelle de snippets et patterns de code réutilisables, au format OKF (fichiers markdown autonomes).
Objectif : orienter la génération de code par LLM (agents de code — Vibe, Junie, WebStorm AI...) sur des patterns déjà validés.

## Contrat pour les agents de code

Ce wiki est écrit **pour vous**. Vous devez le lire comme un standard à appliquer, pas comme de la documentation passive.

### Règle 0 — Zéro commentaire dans le code généré

**Cette règle s'applique à TOUTES les fiches et à TOUT le code généré à partir de ce wiki, quelle que soit la couche (front, backend, infra, patterns).**

- Écrire les actions **de haut en bas**, séparées par des **lignes vides** — un bloc logique par étape. C'est suffisamment lisible, pas de commentaire par-dessus.
- **Jamais** de commentaires descriptifs du type "on fetch les données", "validate input", "// étape 1 : ..." — un commentaire qui répète le code en dessous est du bruit.
- Le **nommage fait le travail** : bien nommer les variables/fonctions (ex. `uploadPromise`, `getUploadDriver()`) remplace la documentation.
- **Exception unique** : un commentaire uniquement quand le code ne peut pas s'expliquer par lui-même — piège non devinable (ex. `stream.resume()` pour drainer un stream rejeté), workaround externe, décision contre-intuitive. Rien d'autre.
- Les commentaires **dans les fiches du wiki** (explications pédagogiques) restent : la règle porte sur le **code généré dans les projets**, pas sur la documentation du wiki.

### Règle 1 — Route d'upload = middleware `uploadValidator` systématique

**Dès que vous identifiez qu'une route reçoit un upload de fichier (multipart/form-data, busboy, input file), vous devez :**

1. **Le signaler à l'utilisateur** (ex. "c'est une route d'upload — j'ajoute le middleware `uploadValidator`, voir la fiche backend/upload-file-busboy.md") ;
2. **Brancher le middleware dans la chaîne** : `router.post(path, uploadValidator, handler)` — jamais de parsing busboy sans validation du Content-Type en amont ;
3. Si le middleware n'existe pas encore dans le projet, **proposer de le créer** avec le pattern de la fiche (rejet du header absent / non-multipart avant tout parsing).

La validation du Content-Type en entrée de chaîne fait partie du standard upload, au même titre que `req.pipe(busboy)` ou le drain des streams — une route d'upload sans `uploadValidator` est une déviation à signaler.

### Autres règles globales

- **Une seule version vivante par besoin** (ex. un seul pattern de pagination). Les anciennes versions partent dans `_archive/` avec la mention "remplacé par X".
- Chaque fiche est autonome : stack, usage, code complet, notes, état.
- `INDEX.md` est le point d'entrée : le seul fichier à charger en priorité, puis uniquement les fiches utiles à la tâche.
- Une fiche couvre le besoin → appliquer son code et ses pièges documentés. **Toute déviation doit être explicitement demandée par l'utilisateur**, jamais décidée seule.
- Erreurs Express : try/catch + `next(err)`, `AppError` avec statut, errorHandler global en dernier (voir `backend/express-error-handling.md`).
- Secrets : jamais en dur, toujours `.env` (voir `infra/docker-secrets-env.md`).
- Range par couche (front/backend/infra/patterns), jamais par projet.

## Structure

- `INDEX.md` — index de toutes les fiches (stack, usage, état vivant/obsolete)
- `front/` — snippets front (Vue 3, Tailwind, DaisyUI, VueUse)
- `backend/` — snippets back / API (Node/Express)
- `infra/` — Docker, déploiement, CI
- `patterns/` — patterns de développement (SOLID, style proto...)
- `stacks.md` — stack de référence + stacks abandonnées
- `_archive/` — snippets remplacés (historique, jamais consultés)
