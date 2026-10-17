---
name: libraries
layer: reference
version: 2
---

# Index des librairies : qui fait quoi

Aide-mémoire rapide des librairies de la chaîne documentaire. Colonne clé : **Node.js / Python** — si une librairie fonctionne mieux (ou uniquement) en Python, elle est candidate au **micro-service Python dédié dans Docker** (backend Python supplémentaire, à part, décidé librairie par librairie).

---

## ocrmypdf — OCR et prétraitement PDF

**Superpose le texte reconnu directement sur les images du PDF original** (layer texte invisible), rendant le texte sélectionnable sans toucher au rendu visuel.

Fonctionnalités :
- **Redressement automatique (deskew)** : remet droit un document scanné de travers
- **Nettoyage des imperfections (unpaper)** : supprime taches, bords noirs de scan, bruit de fond
- **Correction d'orientation** : pivote automatiquement les pages à 90°/180°/270° si à l'envers
- **Compression** : via **pngquant** et **Ghostscript**, réduit considérablement le poids du PDF généré sans perte visible de qualité

Sous le capot : moteur OCR **Tesseract** (à installer séparément).

**Node.js / Python : ❌ Python uniquement** (package Python + CLI). Pas d'équivalent direct appelable depuis Node → candidat micro-service Python. Depuis Node, seul contournement : shell out sur le CLI.

---

## PyMuPDF vs PyMuPDF4LLM — extraction vs Markdown

Deux librairies liées mais distinctes :

- **PyMuPDF** : moteur **bas niveau** qui extrait des éléments bruts — texte filaire, coordonnées cartésiennes (x, y), images, ou du HTML générique peu structuré.
- **PyMuPDF4LLM** : **s'appuie sur PyMuPDF** pour analyser l'agencement visuel (mise en page, détection des titres, tableaux, listes) et reconstituer automatiquement le document en syntaxe Markdown (titres `#`, tableaux `| --- |`, paragraphes).

→ Pour la chaîne pdf-to-md, **toujours PyMuPDF4LLM** : le Markdown structuré est l'entrée requise par le use case anonymisation. PyMuPDF seul servirait si on voulait un accès fin (positions, images extraites).

**Node.js / Python : ❌ Python uniquement** (les deux, PyMuPDF comme PyMuPDF4LLM). Pas de port Node de qualité équivalente → candidats micro-service Python.

---

## LangChain — orchestration LLM

Chaînage de prompts, loaders, modèles. Utilisé dans l'API Node (anonymisation).

**Node.js / Python : ✅ les deux** — équivalents (LangChain.js / LangChain Python). Rester côté Node OK.

---

## Règle de décision (rappel)

- Librairie ✅ Node.js → intégrée directement dans l'API Node.
- Librairie ❌ Python uniquement → micro-service Python séparé dans Docker (une seule fois le pattern établi — pas de fiche dédiée pour l'instant).
