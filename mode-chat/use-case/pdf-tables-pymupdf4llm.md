# PyMuPDF4LLM : extraction des tableaux (find_tables)

**Rôle** : récupérer les tableaux d'un PDF avec leurs cellules, via `page.find_tables()` de PyMuPDF/PyMuPDF4LLM.

## Point clé

Alors que la conversion Markdown complète (`to_markdown`) est jugée non fiable — voir [pdf-to-markown.md](pdf-to-markown.md) — l'extraction de tableaux, elle, **fonctionne bien** :

- `page.find_tables()` renvoie chaque tableau avec ses cellules individuelles ;
- reconstruire un tableau Markdown est trivial : join de `|` sur les cellules de chaque ligne ;
- **aucune IA nécessaire** pour la construction : pur Python, déterministe et rapide ;
- l'IA n'est utile qu'en aval, pour **vérifier/nettoyer** le résultat (cellules fusionnées mal détectées, entêtes multi-lignes…).

## Snippet validé

```python
import pymupdf  # ou pymupdf4llm, même moteur sous-jacent

doc = pymupdf.open("document-source.pdf")

for page in doc:
    for table in page.find_tables():
        for row in table.extract():
            print("| " + " | ".join(cell if cell else "" for cell in row) + " |")
```

Préfixer/suffixer la ligne d'entête avec `|---|---|` selon la première ligne du tableau.

## Liens

- [pdf-to-markown.md](pdf-to-markown.md) : pourquoi `to_markdown` global est écarté, mais `find_tables` garde PyMuPDF4LLM pertinent pour les tableaux.
- [ocr-on-png.md](ocr-on-png.md) : OCR quand le PDF n'a pas de texte sélectionnable.
