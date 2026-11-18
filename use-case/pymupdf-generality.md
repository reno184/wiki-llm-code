# pymupdf (généralité)

## Version Pro (PyMuPDF Pro)

Il existe une **version Pro** : https://pymupdf.readthedocs.io/en/latest/pymupdf-pro/index.html

- Permet entre autres de **charger directement des documents Word/Office**.


> ⚠️ **Faiblesse : licence AGPL** — attention si usage commercial non-libre. Pour commercialiser, il faut une licence commerciale Artifex (payante).

## Positions images / dessins

Voir la fiche dédiée : [pymupdf-images-position.md](pymupdf-images-position.md) — `get_drawings()` donne `"rect"` directement ; `get_images()` ne donne PAS la position, il faut `get_image_info()` (bbox) ou `get_image_rects(xref)`.

## todo

- finir dans le porjet proto selection clip(rect) via dessin rectangle cleu en front

```
Drag sur l'image affichée (clientX/Y + getBoundingClientRect + naturalWidth)
  → coords pixels image (scaleX/scaleY dynamiques)
  → coords espace PDF (× 72/dpi) — stockage normalisé
  → POST API {docId, page, rect: [x0, y0, x1, y1]}
  → micro-service : pymupdf.Rect(x0, y0, x1, y1) + page.get_text(clip=cr)
  → texte extrait retourné au front
```

## État

**vivant** 

## tags
[pymupdf, convert-pdf, python]
