# PyMuPDF — position des images et dessins (bbox / rect)

> ⚠️ **Piège majeur** : `page.get_images()` seul ne suffit PAS pour connaître la position des images sur la page.

## Résumé

| Type | Méthode | Position fournie |
|---|---|---|
| Dessins vectoriels | `page.get_drawings()` | **OUI** — clé `"rect"` directement dans chaque dict |
| Images bitmap | `page.get_images()` | **NON** — liste la ressource brute (xref, résolution, format) sans placement |

Une même image stockée dans le PDF peut être affichée 3 fois à des endroits différents, ou même masquée. `get_images()` ne sait pas où elle est affichée.

## Dessins vectoriels — `page.get_drawings()`

Parcourt les tracés vectoriels (lignes, formes, signatures vectorielles) et fournit **directement la propriété `"rect"`** (rectangle de position sur la page) dans chaque dictionnaire de résultat. Rien d'autre à chercher.

## Images bitmap — obtenir la bbox

### Option A (recommandée) : `page.get_image_info()`

Renvoie les métadonnées de placement des images sur la page avec leur `"bbox"` :

```python
# Contient le "bbox" (rectangle x0, y0, x1, y1)
for img in page.get_image_info():
    bbox = img.get("bbox")  # C'est le rectangle de position sur la page !
```

### Option B : `page.get_image_rects(xref)`

Si vous avez un `xref` d'image issu de `page.get_images()` :

```python
for img in page.get_images():
    xref = img[0]
    rects = page.get_image_rects(xref)  # Liste des positions sur la page
```

## À retenir

* **Dessins vectoriels** : `page.get_drawings()` → `"rect"` directement.
* **Images bitmap** : **indispensablement la `bbox`** via `page.get_image_info()` (ou `get_image_rects(xref)`) pour connaître les coordonnées sur la page.

## Liens croisés

- [pymupdf-generality.md](pymupdf-generality.md) — généralités PyMuPDF
- [pymupdf_tech_anonymisation.md](pymupdf_tech_anonymisation.md) — zones à exclure/anonymiser
- [anonymous-rectangle-clip.md](anonymous-rectangle-clip.md) — rédaction par rectangle

## État

**vivant**

## tags
[pymupdf, get_images, get_drawings, bbox, position, python]
