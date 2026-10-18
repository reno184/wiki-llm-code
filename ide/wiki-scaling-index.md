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

## État

**vivant** — stratégie validée 2026-10, INDEX actuel encore en un seul niveau (trigger hiérarchique pas encore atteint).
