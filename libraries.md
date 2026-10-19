---
name: libraries
layer: reference
version: 10
---

# Index des librairies : qui fait quoi

Aide-mémoire rapide des librairies de la chaîne documentaire. Colonne clé : **Node.js / Python** — si une librairie fonctionne mieux (ou uniquement) en Python, elle est candidate au **micro-service Python dédié dans Docker** (backend Python supplémentaire, à part, décidé librairie par librairie).

---

## pdftocairo (Poppler) — rendu de pages PDF → images

Binaire natif du paquet `poppler-utils` (rendu Cairo, remplace pdftoppm : antialiasing et fidélité supérieurs). Rend les pages en PNG à un dpi donné (`-png -r 150`, `-f/-l` pour un intervalle de pages).

**Rôle (oct. 2026)** : moteur de **rendu d'affichage** de la chaîne — pages → WebP pour l'interface. Code validé : [backend/pdftocairo-sharp.md](backend/pdftocairo-sharp.md). Dépendance Docker : `poppler-utils` (+ `poppler-data` si PDF CJK) dans l'image de l'API Node.

**Node.js / Python : ✅ Node.js** — spawn direct depuis l'API ; les wrappers Python (pdf2image) ne font que lancer le même binaire.

---

## sharp — encodage d'images (WebP) côté Node

Encodage/compression d'images natif (libvips prébuild, npm). Dans la chaîne : PNG pdftocairo → WebP (`sharp(png).webp({quality})`). 30-50% plus léger que PNG à qualité égale.

**Node.js / Python : ✅ Node.js** (npm).

---

## PyMuPDF — extraction de texte ciblée (le rendu d'affichage lui est retiré)

Moteur **bas niveau** qui extrait des éléments bruts — texte filaire, coordonnées cartésiennes (x, y), images. Fiche dédiée : [infra/pymupdf.md](infra/pymupdf.md).

**Décisions gravées (oct. 2026)** :

- La chaîne est **100% PyMuPDF sur PDF natifs**, et l'extraction est **`get_text` uniquement** (déterministe, ciblé, human-in-the-loop) — voir [use-cases/pdf-to-text/README.md](use-cases/pdf-to-text/README.md).
- PyMuPDF n'OCRise pas les PDF image → un scanné est **hors périmètre** (décision produit : pas de contrats scannés).
- **Rendu d'affichage retiré (oct. 2026)** : get_pixmap → WebP pour l'affichage produit passe à pdftocairo + sharp côté Node (voir ci-dessus). get_pixmap garde uniquement son rôle de rendu de **zones vers LLM vision** (use case ocr-llm).

**Node.js / Python : ❌ Python uniquement** → micro-service Python (extraction get_text).

---

## Prettier — formatage Markdown

Formate le Markdown (`npx prettier --write doc.md`) : mise en page, espaces, wrapping, listes, tableaux. Ne reformule pas le texte.

→ Outil disponible, sans use case actif rattaché. **Ne détecte pas les sauts de niveau de titres** : la hiérarchie des titres relève d'une revue IA locale (Ollama), relue.

**Node.js / Python : ✅ Node.js** (npm).

---

## Ollama — LLM local (via llama.cpp)

Runtime de modèles locaux (`llama3.1`, `mistral`, `qwen2.5`…). API HTTP locale + CLI `ollama run`. Dans la chaîne : revue **sémantique** relue (niveaux de titres), jamais de transformation automatique de documents longs.

⚠️ Leçons gravées : un doc long en prompts LLM local = **échec** (reformat-markdown : contrat 7 pages ; pymupdf4llm : hallucination dès la 4e page). Un petit modèle (7–8B) reformule/hallucine → sortie toujours relue, jamais en passe automatique silencieuse.

**Node.js / Python : ✅ les deux** — appel HTTP vers l'API locale Ollama depuis l'API Node, bindings Python existants.

---

## LangChain — orchestration LLM

Chaînage de prompts, loaders, modèles. Utilisé dans l'API Node (anonymisation).

**Node.js / Python : ✅ les deux** — équivalents (LangChain.js / LangChain Python). Rester côté Node OK.

---

## PyMuPDF4LLM — PDF → Markdown automatique (archivé)

Reconstituait le document en Markdown (titres, tableaux, listes) à partir de l'analyse visuelle de PyMuPDF.

⚠️ **Archivé (oct. 2026)** — pipeline complet avec post-traitement sémantique Ollama testé sur un long contrat : **usine à gaz** (boucle convert → Ollama → reconsolidation de chunks, hyper long) et **Qwen a halluciné à partir de la 4ème page**. Décision : **`pymupdf.get_text` uniquement**. Trace et conditions de réveil : [_archive/pymupdf4llm.md](../_archive/pymupdf4llm.md). Le use case pdf-to-md qui l'utilisait est archivé avec lui : [_archive/pdf-to-md.md](../_archive/pdf-to-md.md).

---

## OCRmyPDF — OCR et prétraitement PDF (archivé)

Superposait le texte reconnu (layer invisible) sur les images d'un PDF scanné. Sous le capot : Tesseract.

⚠️ **Archivé (oct. 2026)** — chaîne 100% PyMuPDF sur PDF natifs, périmètre sans scannés → plus besoin de layer OCR amont. Leçons conservées : trace [_archive/python-ocrmypdf.md](../_archive/python-ocrmypdf.md). **Réveil si** besoin réel de scannés : PyMuPDF n'a pas de moteur OCR intégré.

---

## Tesseract OCR & Docling — extraction automatique (archivés)

- **Tesseract (usage autonome)** : moteur OCR bas niveau, résultats décevants sur les documents cibles (contrats, fiches de paie).
- **Docling** : conversion automatique document → structure, même famille que les approches "tout automatique" écartées.

⚠️ **Archivés** — chaîne retenue : **PyMuPDF get_text + ciblage humain via interface Vue.js + human-in-the-loop**. Trace : [_archive/tesseract-docling-standby.md](../_archive/tesseract-docling-standby.md).

---

## pdf2htmlEX — PDF → HTML (archivé)

Binaire C++ de reproduction pixel-perfect. **Positionnement absolu = cause racine des échecs** (chevauchements, sélection bloquée), dépendances obsolètes, déploiement douloureux. Stand-by sur demande client réelle. Trace : [_archive/pdf2htmlex-standby.md](../_archive/pdf2htmlex-standby.md).

---

## Règle de décision (rappel)

- Librairie ✅ Node.js → intégrée directement dans l'API Node.
- Librairie ❌ Python uniquement → micro-service Python séparé dans Docker.
