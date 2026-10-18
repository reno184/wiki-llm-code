---
use-case: correction-syntax-markdown
version: 1
---

# Use case : correction de syntaxe Markdown (correction-syntax-markdown)

Passe de contrôle qualité sur un fichier Markdown : corrige la hiérarchie des titres (niveaux H1–H6, sauts de niveau, H1 multiples) et la mise en forme. Se branche entre [pdf-to-md](../pdf-to-md/README.md) et [anonymisation](../anonymisation/README.md).

## Principe : déterministe d'abord, IA locale en complément

1. **Passes déterministes** (mdformat) : reproductibles, ne reformulent pas le texte, corrigent ce qui est mécanique (mise en forme, espaces, wrapping, tableaux).
2. **Passes IA locale** (Ollama) : hiérarchie des titres et tout ce qui demande du jugement sémantique (ex. un H3 « Conclusion » qui devrait être un H2, sauts de niveau, H1 multiples). Un LLM local peut halluciner → jamais en passe automatique, sortie relue.

## Chaîne de traitement

```
Markdown source (sortie pdf-to-md)
   │
   ▼
mdformat                 ← 1. normalisation de la mise en forme (espaces, wrapping,
   │                        listes, tableaux) — ne reformule pas le texte
   ▼
Ollama (optionnel)       ← 2. revue sémantique des niveaux de titres (sauts de niveau,
   │                        H1 multiples, ponctuation de fin) — relue humaine
   ▼
Markdown validé → use-cases/anonymisation/
```

## Commandes

```bash
# 1. Normaliser la mise en forme (Python)
pip install mdformat
mdformat doc.md

# 2. Revue sémantique des titres avec un LLM local (optionnel, sortie à relire)
ollama run llama3.1 "Corrige la hiérarchie des titres de ce markdown sans reformuler le texte : $(cat doc.md)"
```

## Règles de titres à faire respecter (par la passe IA locale, relue)

- Un seul H1 par document.
- Aucun saut de niveau (H2 → H4 interdit).
- Un titre ne se termine pas par une ponctuation.
- Les niveaux reflètent la logique du document, pas la taille visuelle.

## Fiches liées

- [libraries.md](../../libraries.md) — mdformat, Prettier, Ollama : qui fait quoi

## Règle de synchronisation

Ce use case est un **contrôle qualité** de la chaîne documentaire : tout Markdown produit par pdf-to-md doit passer la passe mdformat avant anonymisation. La passe IA locale corrige la hiérarchie des titres, en relecture humaine obligatoire.
