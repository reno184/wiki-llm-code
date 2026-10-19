# Conversion d'échelle image ↔ espace PDF

Fiche technique du calcul de coordonnées du use case [pdf-to-text](../../use-cases/pdf-to-text/README.md) — math pure, testable isolément. Décisions produit : [cartographie-rect-image.md](../../use-cases/pdf-to-text/cartographie-rect-image.md).

## La double conversion

**1. Affiché → pixels image** (échelle dynamique, jamais supposée à 1 — l'image est redimensionnée par CSS) :

```javascript
const rect = imageElement.getBoundingClientRect();
const scaleX = image.naturalWidth / rect.width;
const x_image = (e.clientX - rect.left) * scaleX;
```

**2. Pixels image → espace PDF** (PyMuPDF : points à 72 dpi, origine en haut à gauche, y vers le bas) :

```javascript
const scalePdf = 72 / dpi;
const x_pdf = x_image * scalePdf;
```

## Règles

- **Stockage normalisé en espace PDF** — jamais en pixels : les Rects survivent au changement de niveau (dpi150 → dpi300), à la taille du conteneur, au devicePixelRatio (Retina).
- L'échelle est **recalculée à chaque interaction** (conteneur CSS redimensionnable, `useElementSize` VueUse pour réagir).
- Conversion inverse (PDF → image) uniquement au rendu, pour surligner des zones enregistrées. La formule `x_image = x_pdf / (72/dpi)` puis division par scaleX.
- `dpi` = le dpi de L'IMAGE DESSINÉE : le ciblage se fait sur le dpi300 (surface de ciblage standard), le dpi150 est la navigation — même formule, `dpi` different.

## Piège notoire

L'espace PyMuPDF est en **points à 72 dpi**, pas en pixels — le facteur `dpi/72` est la source d'erreur n°1 dès qu'on mélange pixmap et Rects (documenté dans [infra/pymupdf.md](../../infra/pymupdf.md)).

## Rotation : hors périmètre

Décision produit : l'avocat ne travaille pas sur des contrats scannés → pas de gestion de `page.rotation`. Un document avec pages tournées est hors périmètre.

## État

**vivant** — formules actées (oct. 2026), à consolider avec les tests du prototype (le jour du dpi300, la formule `dpi=300` doit se substituer sans changement de code).
