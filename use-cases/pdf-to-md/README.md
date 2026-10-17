---
use-case: pdf-to-md
version: 1
---

# Use case : PDF → Markdown (pdf-to-md)

Convertit des PDF (scannés ou natifs) en Markdown exploitables par le use case [anonymisation](../anonymisation/README.md).

## Chaîne de traitement

```
PDF source
   │
   ▼
ocrmypdf                  ← 1. prétraitement (voir ocrmypdf.md : les 2 rôles)
   │
   ▼
PyMuPDF4LLM (pymupdf4llm) ← 2. conversion → Markdown
   │
   ▼
Markdown structuré → use-cases/correction-syntax-markdown/ (contrôle qualité titres)
   │
   ▼
Markdown validé → use-cases/anonymisation/
```

## Pourquoi OCRmyPDF systématiquement avant PyMuPDF4LLM

OCRmyPDF joue **2 rôles** dans la chaîne :

1. **Ajout d'un layer texte invisible** sur les PDF scannés (image seule) : le texte devient sélectionnable, donc extractible par PyMuPDF4LLM. Sans ce layer, PyMuPDF4LLM ne sort rien d'exploitable sur un scan.
2. **Optimisation/nettoyage du PDF** avant conversion : meilleure qualité d'extraction en sortie.

→ Toujours passer par OCRmyPDF, même si le PDF paraît « textuel » : le coût est faible et ça garantit un layer texte propre pour PyMuPDF4LLM.

## Fiches liées
- [ocrmypdf.md](ocrmypdf.md) — commandes OCRmyPDF (2 use cases : layer texte invisible + optimisation)
- [pymupdf4llm.md](pymupdf4llm.md) — conversion Markdown via PyMuPDF4LLM

## Règle de synchronisation
La sortie de ce pipeline (Markdown) passe par le use case [correction-syntax-markdown](../correction-syntax-markdown/README.md) (contrôle qualité des titres, markdownlint --fix obligatoire) avant d'être l'entrée **exclusive** du use case anonymisation : format d'entrée = `text/markdown`, pas de PDF direct.
