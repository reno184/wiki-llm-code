# Wiki-LLM scaling — INDEX hiérarchique et périmètre par projet

Stack: repo wiki-llm-code (GitHub), MCP github, agents IDE (Junie/Vibe), project rules.
Usage: garder le wiki utilisable par les agents quand il atteint des dizaines/centaines de fiches — routage léger, périmètre restreint par projet, hygiène anti-inflation.

## Problème

Un INDEX.md unique qui atteint des centaines de lignes est contre-productif : l'agent le charge à chaque tâche (contexte noyé) et ne filtre pas naturellement les fiches hors périmètre du projet.

## Solution 1 — INDEX racine = routeur léger

- **INDEX.md racine** : une ligne par fiche (stack, usage en 5-10 mots, état). Rien de plus. Pas de description longue : c'est un routeur, pas une doc.
- L'agent lit l'INDEX (léger), identifie la fiche via la description, charge ensuite la fiche seule. Jamais deux fiches de chargées pour une seule question de routage.

## Solution 2 — INDEX hiérarchique (au-delà de ~20-25 fiches par répertoire)

```
INDEX.md            → routeur racine : une ligne par fiche + lien vers sous-index des gros répertoires
backend/INDEX.md    → détail des fiches backend
python/INDEX.md     → si backend/ explose encore (2e niveau)
```

- Règle projet : "charge INDEX.md, puis le sous-index du répertoire concerné si l'INDEX racine pointe dessus". L'agent ne charge jamais plus de 2 fichiers d'index.
- Se déclencher sur ~20-25 fiches dans un même répertoire, pas avant (sinon indirection inutile).

## Solution 3b — Profils projet (agents first : routage par contexte, pas par couche)

L'agent charge selon le **contexte du projet**, pas selon la couche technique. Un profil = une vue sur le wiki, pas un rangement :

- Une fiche peut être référencée dans **plusieurs profils** (ex. nodejs-express-env dans Site vitrine ET SPA+API) — elle reste UNE seule fiche (source de vérité unique).
- Une fiche transversale (ex. cors.md : concerne API et multipage) n'est PAS dupliquée : elle existe une fois, les profils la référencent selon le besoin.

Structure : section "Profils projet" en tête d'INDEX.md, un profil par type de projet (Site vitrine, SPA+API, Script Python). Le project rules de chaque projet désigne son profil :

```markdown
- Profil wiki : **Site vitrine**. Charge INDEX.md puis uniquement
  les fiches listées sous ce profil.
```

Le test de cohérence : corriger un sujet (ex. config CORS) ne doit toucher qu'UN fichier. Deux fichiers à modifier = la structure a échoué.

## Solution 3 — Périmètre par projet (dans le project rules, pas des tags)

Formulation type pour un projet Python + Docker :

```markdown
- AVANT de coder : charge INDEX.md du repo reno184/wiki-llm-code via
  MCP GitHub (outil get_file_contents). Ce projet n'utilise que Python
  et Docker : ne charger que les fiches des répertoires backend/,
  infra/ et patterns/ listées dans l'INDEX. Ignorer front/ et ide/.
```

Pourquoi répertoires plutôt que tags :

- **Zéro mécanisme à maintenir** : les tags exigent un frontmatter strict sur chaque fiche + un parsing de métadonnées par l'agent (étape en plus = échec en plus). Les répertoires existent déjà.
- **L'INDEX fait office de filtre** : l'agent lit un INDEX léger, voit les fiches du périmètre, ignore le reste. Même effet que le tag, une lecture de moins.
- La règle dit "par défaut", pas "interdit" : une fiche hors périmètre reste chargeable si un besoin réel émerge.

## Anti-patterns

- **search_code GitHub comme routage primaire** : résultats bruités, plus cher qu'un INDEX structuré. À réserver au debug "où ai-je parlé de X ?".
- **Tags frontmatter** : mécanisme à maintenir + parsing fragile côté agent.
- **Descriptions longues dans l'INDEX** : noient le routage, le détail vit dans la fiche.
- **INDEXer les archives** : l'INDEX ne référence que l'état `vivant`. `_archive/` reste hors index (sinon l'agent charge des fiches obsolètes).

## Hygiène anti-inflation

- Une seule version vivante par besoin, anciennes versions → `_archive/`, jamais re-documentées.
- Fiche autonome : si une fiche dépasse ~150 lignes, elle couvre probablement deux besoins → la couper en deux.



## Maintenance — signaux de remaniement (passe légère continue)

Remaniement au fil de l'eau, jamais en grand chantier. À chaque fois qu'on touche une fiche, 30 secondes de vérification (règle boy scout). Signaux :

- Hésiter 5 s sur "où ranger cette fiche ?" → répertoire peu clair ou fiche couvrant deux besoins.
- Un agent charge la mauvaise fiche, ou deux fiches disent presque la même chose → duplication, fusionner ou archiver.
- Une ligne d'INDEX dépasse ~15 mots → la fiche a trop de responsabilités, la couper.
- "C'est un peu dans ce coin-là aussi" → frontière floue entre répertoires, redécouper.
- Une fiche dépasse ~150 lignes → probablement deux besoins, couper en deux.

Règle de découpe : **une fiche = une raison de changer** (Single Responsibility). Deux sujets qui évoluent toujours ensemble → fusionner ; séparément mais dans la même fiche → couper.

Passe mensuelle légère (5 min) : relire l'INDEX, vérifier les états `vivant`, traquer les doublons, mettre à jour les profils si les fiches ont bougé.

## État

**vivant** — stratégie validée 2026-10. INDEX avec section Profils projet (Site vitrine, SPA+API, Script Python) + ligne backend/cors.md. INDEX hiérarchique pas encore nécessaire (trigger ~20-25 fiches/répertoire pas atteint).
