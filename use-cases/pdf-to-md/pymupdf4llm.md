---
use-case: pdf-to-md
layer: backend-python
version: 1
---

# PyMuPDF4LLM : conversion Markdown

## Contexte d'usage
Étape 2 de la chaîne pdf-to-md : reçoit le PDF prétraité par OCRmyPDF et produit le Markdown.

## PyMuPDF vs PyMuPDF4LLM

- **PyMuPDF** : moteur bas niveau — extrait texte filaire, coordonnées cartésiennes (x, y), images, ou HTML générique peu structuré.
- **PyMuPDF4LLM** : s'appuie sur PyMuPDF et analyse l'agencement visuel (mise en page, titres, tableaux, listes) pour reconstituer le document en Markdown (titres `#`, tableaux `| --- |`, paragraphes).

→ Toujours **PyMuPDF4LLM** pour ce use case : le Markdown structuré est l'entrée requise par l'anonymisation.

## Utilisation de base

```python
import pymupdf4llm

md = pymupdf4llm.to_markdown("doc_pretraite.pdf")
with open("doc.md", "w", encoding="utf-8") as f:
    f.write(md)
```

## Options utiles (à compléter selon usage réel)

- `page_chunks` : sortie par page (utile si le chunking anonymisation se fait par page)
- `ocr_language` : OCR multi-langues pour les documents multilingues (voir ci-dessous)
- extraction des tables, headers : à documenter si utilisés

## Traduction / documents multilingues (ocr_language)

`to_markdown` accepte un paramètre `ocr_language` (codes Tesseract) pour OCRiser un document multilingue :

```python
import pymupdf4llm

md_text = pymupdf4llm.to_markdown("multilingual.pdf",
                                   ocr_language="eng+deu")
```

- Le moteur OCR sous-jacent est **Tesseract** : il faut les packs de langue installés sur la machine/conteneur.
- Codes séparés par `+` (ex. `eng+deu`, `fra+eng`).

### Installation des packs Tesseract

**Hôte Debian/Ubuntu** :

```bash
sudo apt install tesseract-ocr-deu tesseract-ocr-fra
```

**En Docker** (adaptation, cf. infra/dockerfile-python.md) : les packs de langue sont des **paquets système**, pas des paquets pip — ils s'installent dans le Dockerfile, PAS dans requirements.txt :

```dockerfile
FROM python:3.12-slim

RUN apt-get update \
    && apt-get install -y --no-install-recommends tesseract-ocr tesseract-ocr-deu tesseract-ocr-fra \
    && rm -rf /var/lib/apt/lists/*
```

- `requirements.txt` (pip) : uniquement `pymupdf4llm` (+ `ocrmypdf` selon la chaîne). Jamais les packs Tesseract dedans : ils ne sont pas sur PyPI au sens installable.
- Installer les paquets apt **avant** le layer pip pour ne pas casser le cache requirements.
- Nettoyage `apt lists` dans le même RUN (image minimale, règle --no-cache-dir).

## Doc officielle

- https://pymupdf.readthedocs.io/en/latest/pymupdf4llm/ — référence PyMuPDF4LLM (options `to_markdown`, `page_chunks`, `ocr_language`, tables, headers).

## Notes
- Prérequis : le PDF a toujours un layer texte (garanti par OCRmyPDF en amont).
- Sortie = Markdown structuré → entrée du use case anonymisation.
- À compléter avec les options réelles validées.
