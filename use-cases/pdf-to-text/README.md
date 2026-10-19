---
use-case: pdf-to-text
version: 2
---

# Use case : PDF → texte ciblé, semi-automatique (pdf-to-text)

Extraction de texte **ciblée** depuis un PDF, avec **ciblage humain dans une interface Vue.js** et **human-in-the-loop systématique**. Produit : texte exploitable de zones sélectionnées — pas du Markdown intégral, pas du 100% automatique.

Cible produit : avocats / cabinets M&A / DRH — contrats, fiches de paie. Voir [business-idea/ia-legaltech-contrat.md](../../business-idea/ia-legaltech-contrat.md).

## Architecture

```
PDF source (natif, ou scanné passé par OCRmyPDF)
   │
   ▼
Rendu d'affichage : pages → WebP (dpi150 navigation / dpi300 ciblage) + blurhash
   │        → détail : [affichage-webp.md](affichage-webp.md)
   ▼
Interface Vue.js : mode ciblage sur page figée, drag de zones sur l'image dpi300
   │        → décisions : [cartographie-rect-image.md](cartographie-rect-image.md)
   │        → composant : [../../front/javascript/draw-rectangle.md](../../front/javascript/draw-rectangle.md)
   ▼
API : extraction ciblée via PyMuPDF (clip, search_for, find_tables, get_drawings)
   │        → socle : [../../infra/pymupdf.md](../../infra/pymupdf.md)
   ▼
Texte extrait des zones → prévisualisation / correction dans l'interface
   │
   ▼
Validation humaine (human-in-the-loop) → texte exploitable
```

## Pourquoi semi-automatique

- La transformation 100% automatique des documents longs (contrats) est un échec constaté : LLM local (reformatage, échec contrat 7 pages), pdf2htmlEX (archivé), Tesseract autonome et Docling (archivés).
- La fiabilité vient du **ciblage humain** : l'utilisateur désigne ce qui compte, PyMuPDF sort le texte de ces zones de façon déterministe.
- Le human-in-the-loop est un **choix d'architecture** (niveau d'exigence juridique), pas un pis-aller.

## Décisions actées (oct. 2026)

- Rendu d'affichage WebP : pyramide blurhash (base) / dpi150 (navigation) / dpi300 (surface de ciblage) — [affichage-webp.md](affichage-webp.md).
- Ciblage : drag + poignées, échelle dynamique, Rects stockés en espace PDF normalisé, page figée (jamais pendant un motion), rotation hors périmètre — [cartographie-rect-image.md](cartographie-rect-image.md).
- Composant front : trajectoire P1 100% VueUse → P2 composant Vue étanche → P3 D3 modulaire (d3-drag + d3-scale) ; Web Component supprimé — [../../front/javascript/draw-rectangle.md](../../front/javascript/draw-rectangle.md).
- Moteur : micro-service Python Docker (PyMuPDF) — unicité du moteur rendu + extraction pour garantir image affichée = coordonnées d'extraction.

## Périmètre / hors périmètre

- **Ce use case** : le workflow ciblage → extraction → validation (l'interface, l'API, les patterns d'extraction par type de document : clauses, grilles de paie, tableaux sans bordures).
- **Socle technique PyMuPDF** (méthodes, pièges) : [infra/pymupdf.md](../../infra/pymupdf.md) — ne pas dupliquer ici.
- **Conversion intégrale PDF → Markdown** (brouillon pour relecture) : [../pdf-to-md/README.md](../pdf-to-md/README.md).
- **Notes manuscrites / documents courts** (vision LLM) : [../ocr-llm/README.md](../ocr-llm/README.md) — autre chaîne, autre zone de validité.

## Points ouverts (à traiter en profondeur)

- Ergonomie du ciblage Vue.js : prévisualisation du texte extrait, correction inline, retour visuel (différé — décision 5).
- Contrat d'API : format d'entrée (Rect par zone), format de sortie (texte brut par zone, table extrait, mixte) — question reportée (prototype en cours).
- Stockage des zones et multi-zones : reportés (prototype en cours).
- Patterns par type de document : grilles de paie (find_tables), clauses (search_for + clip), tableaux sans bordures (get_drawings / cluster_drawings).
- Génération WebP : nommage ({docId}/p{NNN}_{level}.webp), lazy loading fenêtré (±3 pages dpi150), dpi300 à la demande avec cache, invalidation par hash du PDF.
- Prétraitement des scannés : OCRmyPDF en amont (layer texte) avant extraction ciblée.

## Fiches liées

- [affichage-webp.md](affichage-webp.md) — couche d'affichage (WebP + blurhash)
- [cartographie-rect-image.md](cartographie-rect-image.md) — décisions ciblage
- [../../front/javascript/draw-rectangle.md](../../front/javascript/draw-rectangle.md), [../../front/javascript/event-delegation.md](../../front/javascript/event-delegation.md), [../../front/javascript/coord-scale-conversion.md](../../front/javascript/coord-scale-conversion.md) — fiches techniques chargées par les agents de coding
- [../../infra/pymupdf.md](../../infra/pymupdf.md) — socle technique
- [../pdf-to-md/README.md](../pdf-to-md/README.md) — conversion intégrale (l'autre moitié de la paire)
- [../anonymisation/README.md](../anonymisation/README.md) — aval potentiel du texte extrait
- [../../business-idea/ia-legaltech-contrat.md](../../business-idea/ia-legaltech-contrat.md) — cible produit

## État

**en construction** — architecture complète cadrée (affichage WebP + ciblage + micro-service) ; prototype P1 (composant VueUse) à développer, questions API/stockage/multi-zones reportées.
