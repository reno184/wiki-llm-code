---
use-case: pdf-to-md
layer: backend-python
version: 0 (squelette)
---

# PyMuPDF4LLM : conversion Markdown

## Contexte d'usage
Étape 2 de la chaîne pdf-to-md : reçoit le PDF prétraité par OCRmyPDF et produit le Markdown.

## Utilisation de base

```python
import pymupdf4llm

md = pymupdf4llm.to_markdown("doc_pretraite.pdf")
with open("doc.md", "w", encoding="utf-8") as f:
    f.write(md)
```

## Options utiles (à compléter selon usage réel)

- `page_chunks` : sortie par page (utile si le chunking anonymisation se fait par page)
- extraction des tables, headers : à documenter si utilisés

## Notes
- Prérequis : le PDF a toujours un layer texte (garanti par OCRmyPDF en amont).
- Sortie = Markdown structuré → entrée du use case anonymisation.
- À compléter avec les options réelles validées.
