---
use-case: pdf-to-text
version: 1
---

# Use case : PDF → texte ciblé, semi-automatique (pdf-to-text)

Extraction de texte **ciblée** depuis un PDF, avec **ciblage humain dans une interface Vue.js** et **human-in-the-loop systématique**. Produit : texte exploitable de zones sélectionnées — pas du Markdown intégral, pas du 100% automatique.

Cible produit : avocats / cabinets M&A / DRH — contrats, fiches de paie. Voir [business-idea/ia-legaltech-contrat.md](../../business-idea/ia-legaltech-contrat.md).

## Architecture

```
PDF source (natif, ou scanné passé par OCRmyPDF)
   │
   ▼
Interface Vue.js : l'utilisateur sélectionne les zones (Rect)
   │
   ▼
API : extraction ciblée via PyMuPDF (clip, search_for, find_tables, get_drawings)
   │
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

## Périmètre / hors périmètre

- **Ce use case** : le workflow ciblage → extraction → validation (l'interface, l'API, les patterns d'extraction par type de document : clauses, grilles de paie, tableaux sans bordures).
- **Socle technique PyMuPDF** (méthodes, pièges) : [infra/pymupdf.md](../../infra/pymupdf.md) — ne pas dupliquer ici.
- **Conversion intégrale PDF → Markdown** (brouillon pour relecture) : [../pdf-to-md/README.md](../pdf-to-md/README.md).
- **Notes manuscrites / documents courts** (vision LLM) : [../ocr-llm/README.md](../ocr-llm/README.md) — autre chaîne, autre zone de validité.

## Points ouverts (à traiter en profondeur)

- Ergonomie du ciblage Vue.js : sélection de zones, prévisualisation du texte extrait, correction inline.
- Contrat d'API : format d'entrée (Rect par zone), format de sortie (texte brut par zone, table extrait, mixte).
- Patterns par type de document : grilles de paie (find_tables), clauses (search_for + clip), tableaux sans bordures (get_drawings / cluster_drawings).
- Où tourne PyMuPDF : micro-service Python Docker (librairies ❌ Node, voir [libraries.md](../../libraries.md)) — à trancher.
- Prétraitement des scannés : OCRmyPDF en amont (layer texte) avant extraction ciblée.

## Fiches liées

- [../../infra/pymupdf.md](../../infra/pymupdf.md) — socle technique
- [../pdf-to-md/README.md](../pdf-to-md/README.md) — conversion intégrale (l'autre moitié de la paire)
- [../anonymisation/README.md](../anonymisation/README.md) — aval potentiel du texte extrait
- [../../business-idea/ia-legaltech-contrat.md](../../business-idea/ia-legaltech-contrat.md) — cible produit

## État

**en construction** — squelette créé (oct. 2026), sujet à traiter en profondeur : ergonomie ciblage, patterns par type de document, contrat d'API.
