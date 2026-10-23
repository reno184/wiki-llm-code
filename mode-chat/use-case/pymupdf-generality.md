# pymupdf (généralité)

## Version Pro (PyMuPDF Pro)

Il existe une **version Pro** : https://pymupdf.readthedocs.io/en/latest/pymupdf-pro/index.html

- Permet entre autres de **charger directement des documents Word/Office**.


> ⚠️ **Faiblesse : licence AGPL** — attention si usage commercial non-libre. Pour commercialiser, il faut une licence commerciale Artifex (payante).

## todo

- voir la méthode get_drawings (extraire tableau sans bordure ? demander à l'ia je ne comprends pas)
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