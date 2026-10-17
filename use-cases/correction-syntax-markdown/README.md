---
use-case: correction-syntax-markdown
version: 0 (squelette)
---

# Use case : correction de syntaxe Markdown (correction-syntax-markdown)

Passe de contrôle qualité sur un fichier Markdown : corrige la hiérarchie des titres (niveaux H1–H6, sauts de niveau, H1 multiples) et la mise en forme. Se branche entre [pdf-to-md](../pdf-to-md/README.md) et [anonymisation](../anonymisation/README.md).

## Principe : déterministe d'abord, IA locale en complément

1. **Passes déterministes** (markdownlint / mdformat) : reproductibles, ne reformulent pas le texte, corrigent ce qui est mécanique.
2. **Passes IA locale** (Ollama) : uniquement pour ce qui demande du jugement sémantique (ex. un H3 « Conclusion » qui devrait être un H2). Un LLM local peut halluciner → jamais en passe automatique, sortie relue.

## Chaîne de traitement

```
Markdown source (sortie pdf-to-md)
   │
   ▼
markdownlint-cli --fix   ← 1. détection + correction auto : MD001 (sauts de niveau),
   │                        MD025 (H1 multiples), MD026 (ponctuation de fin)…
   ▼
mdformat                 ← 2. normalisation de la mise en forme (optionnel)
   │
   ▼
Ollama (optionnel)       ← 3. revue sémantique des niveaux mal choisis — relue humaine
   │
   ▼
Markdown validé → use-cases/anonymisation/
```

## Commandes

```bash
# 1. Détecter les problèmes de titres
npx markdownlint-cli doc.md

# 2. Corriger ce qui est corrigeable automatiquement
npx markdownlint-cli --fix doc.md

# 3. Normaliser la mise en forme (optionnel, Python)
pip install mdformat
mdformat doc.md

# 4. Revue sémantique avec un LLM local (optionnel, sortie à relire)
ollama run llama3.1 "Corrige la hiérarchie des titres de ce markdown sans reformuler le texte : $(cat doc.md)"
```

## Règles de titres à faire respecter

- Un seul H1 par document.
- Aucun saut de niveau (H2 → H4 interdit).
- Un titre ne se termine pas par une ponctuation.
- Les niveaux reflètent la logique du document, pas la taille visuelle.

## Fiches liées

- [libraries.md](../../libraries.md) — markdownlint, mdformat, Prettier, Ollama : qui fait quoi

## Règle de synchronisation

Ce use case est un **contrôle qualité** de la chaîne documentaire : tout Markdown produit par pdf-to-md doit passer la passe markdownlint --fix (erreurs résiduelles = bloquantes) avant anonymisation. La passe IA locale reste optionnelle et relue.
