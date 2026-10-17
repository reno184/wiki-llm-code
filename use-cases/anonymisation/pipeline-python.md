---
use-case: anonymisation
layer: backend-python
version: 1
---

# Pipeline Python : anonymisation

## Contexte d'usage
Fonctions Python du workflow anonymisation : découpage du Markdown, construction de la mapping table, appel LLM, réassemblage. Entrée = Markdown produit par [pdf-to-md](../pdf-to-md/README.md).

## Stratégie de chunking (décision)

**Page comme unité primaire, tokens comme garde-fou.**

- `page_chunks=True` côté PyMuPDF4LLM → chaque page = un chunk (frontières naturelles, pas de tableau coupé).
- Sous-découpage par tokens **uniquement si une page dépasse max_tokens** (rare : page très dense type annexe comptable).

Pourquoi pas par tokens partout : découpage complexe à écrire, risque de couper paragraphes/tableaux, et ne résout pas mieux la cohérence des tags. Pourquoi pas par page seule : une page peut dépasser la fenêtre de contexte.

## Architecture 2 passes (cohérence des tags)

Le prompt exige `[NOM_1]` identique sur tout le document (règle 3). Chaque chunk traité indépendamment ne peut pas garantir ça → il faut un état partagé.

### Passe 1 : mapping table globale
Appel LLM (rapide) sur le document (ou premières pages si trop long) → construit la **mapping table globale** : « Acme SAS → [SOCIETE_1], Dupont → [NOM_1] ».
- Fusion incrémentale si passe 1 aussi découpée : chaque chunk enrichit la table du précédent.
- Les entités sont quasi toujours introduites en page 1 et répétées ensuite (contrats, documents administratifs).

### Passe 2 : anonymisation
Chaque page anonymisée **avec la mapping table injectée dans le prompt** (`{document}` + `{mapping}`) : le LLM applique la table existante au lieu de réinventer des tags.

## Fonctions

```python
def load_document(path: str) -> str:
    """Charge le Markdown (utf-8), vérifie non vide."""

def chunk_document(pages: list[str], max_tokens: int) -> list[str]:
    """Chunk par page (page_chunks=True en amont).
    Sous-découpage par tokens UNIQUEMENT si une page dépasse max_tokens."""

def build_mapping(pages: list[str]) -> dict[str, str]:
    """Passe 1 : LLM → mapping table globale {entité: tag}.
    Fusion incrémentale si le document dépasse le contexte."""

def anonymize_chunk(page: str, mapping: dict[str, str]) -> str:
    """Passe 2 : LLM avec {document} + {mapping} injectés."""

def reassemble(chunks: list[str]) -> str:
    """Réassemble les pages anonymisées dans l'ordre,
    mapping table globale en annexe du document final."""
```

## Impact prompt

La passe 2 injecte `{mapping}` : le prompt canonique `prompts/anonymisation.md` devra ajouter cette variable (bump version) — à faire lors de l'implémentation, pas maintenant.

## Notes
- Points durs : fusion de mapping entre chunks (contradictions), pages dépassant le contexte.
- À compléter avec le code réel validé.
