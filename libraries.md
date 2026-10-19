---
name: libraries
layer: reference
version: 8
---

# Index des librairies : qui fait quoi

Aide-mémoire rapide des librairies de la chaîne documentaire. Colonne clé : **Node.js / Python** — si une librairie fonctionne mieux (ou uniquement) en Python, elle est candidate au **micro-service Python dédié dans Docker** (backend Python supplémentaire, à part, décidé librairie par librairie).

---

## PyMuPDF vs PyMuPDF4LLM — extraction vs Markdown (chaîne retenue, 100% PyMuPDF)

Deux librairies liées mais distinctes :

- **PyMuPDF** : moteur **bas niveau** qui extrait des éléments bruts — texte filaire, coordonnées cartésiennes (x, y), images, ou du HTML générique peu structuré. Fiche dédiée : [infra/pymupdf.md](infra/pymupdf.md) — c'est le socle de TOUTE la chaîne retenue : extraction ciblée (contrats, fiches de paie : ciblage humain via interface Vue.js + clip/search_for/find_tables + human-in-the-loop), rendu d'affichage (get_pixmap → WebP), manipulation PDF.
- **PyMuPDF4LLM** : **s'appuie sur PyMuPDF** pour analyser l'agencement visuel (mise en page, détection des titres, tableaux, listes) et reconstituer automatiquement le document en syntaxe Markdown (titres `#`, tableaux `| --- |`, paragraphes).

→ Pour la chaîne pdf-to-md : **toujours PyMuPDF4LLM** (le Markdown structuré est l'entrée requise par l'anonymisation) ; PyMuPDF seul pour l'accès fin (positions, extraction ciblée, images extraites).

**Décision (oct. 2026) : la chaîne est 100% PyMuPDF sur PDF natifs** — plus aucune OCR dans le pipeline (ocrmypdf archivé). PyMuPDF n'OCRise pas les PDF image → un scanné est **hors périmètre** (cohérent avec la décision produit : pas de contrats scannés).

**Node.js / Python : ❌ Python uniquement** (les deux). Pas de port Node de qualité équivalente → candidats micro-service Python.

---

## Prettier — formatage Markdown

Formate le Markdown (`npx prettier --write doc.md`) : mise en page, espaces, wrapping, listes, tableaux. Ne reformule pas le texte.

→ Outil disponible, sans use case actif rattaché (le use case correction-syntax-markdown est archivé, trace : _archive/correction-syntax-markdown-standby.md). **Ne détecte pas les sauts de niveau de titres** : la hiérarchie des titres relève d'une revue IA locale (Ollama), relue.

**Node.js / Python : ✅ Node.js** (npm).

---

## Ollama — LLM local (via llama.cpp)

Runtime de modèles locaux (`llama3.1`, `mistral`, `qwen2.5`…). API HTTP locale + CLI `ollama run`. Dans la chaîne : revue **sémantique** des niveaux de titres (un titre logiquement H2 resté H3, sauts de niveau, H1 multiples) — disponible à la demande, sans use case actif rattaché.

⚠️ Un petit modèle local (7–8B) peut reformuler ou halluciner → sortie toujours relue, jamais en passe automatique silencieuse.

**Node.js / Python : ✅ les deux** — appel HTTP vers l'API locale Ollama depuis l'API Node, bindings Python existants.

---

## LangChain — orchestration LLM

Chaînage de prompts, loaders, modèles. Utilisé dans l'API Node (anonymisation).

**Node.js / Python : ✅ les deux** — équivalents (LangChain.js / LangChain Python). Rester côté Node OK.

---

## OCRmyPDF — OCR et prétraitement PDF (archivé)

Superposait le texte reconnu (layer invisible) sur les images d'un PDF scanné — deskew, nettoyage (unpaper), orientation, compression (pngquant, Ghostscript). Sous le capot : Tesseract.

⚠️ **Archivé (oct. 2026)** — la chaîne retenue est **100% PyMuPDF sur PDF natifs**, et le périmètre produit exclut les scannés (pas de contrats scannés côté avocat) → plus besoin de layer OCR amont. Leçons conservées : trace [_archive/python-ocrmypdf.md](../_archive/python-ocrmypdf.md) (gonflement de taille : skip-text > redo_ocr > jamais force_ocr par défaut). **Réveil si** un besoin réel de scannés apparaît : PyMuPDF n'a pas de moteur OCR intégré, c'est OCRmyPDF (ou équivalent) qu'il faudra réintroduire.

---

## Tesseract OCR & Docling — extraction automatique (archivés)

- **Tesseract (usage autonome)** : moteur OCR bas niveau, résultats décevants sur les documents cibles (contrats, fiches de paie).
- **Docling** : conversion automatique document → structure, même famille que les approches "tout automatique" écartées — la fiabilité vient du ciblage humain, pas de la devinette automatique.

⚠️ **Archivés** — chaîne retenue à la place : **PyMuPDF + ciblage humain via interface Vue.js + human-in-the-loop** (fiche infra/pymupdf.md). Pas de process 100% automatique visé pour contrats/fiches de paie. Trace : [_archive/tesseract-docling-standby.md](../_archive/tesseract-docling-standby.md).

---

## pdf2htmlEX — PDF → HTML (archivé)

Binaire C++ qui convertit des PDF en documents HTML en cherchant la reproduction pixel-perfect de la mise en page : mêmes polices, mêmes espacements, positionnement absolu des éléments. Exécutable depuis un backend Node.js, Python ou tout environnement supportant les appels système — l'interface paraît simple, c'est le piège.

⚠️ **Archivé** — désavantages en chaîne complète :

- **Positionnement absolu = cause racine des échecs** : les lignes chevauchées et la sélection bloquée dans les tableaux ne sont pas des bugs accidentels, c'est l'architecture même de l'outil. Un retravailillage du HTML généré ne donnera probablement jamais une sélection fluide.
- **Fidélité ~95 %** : lignes chevauchées incurables par paramétrage (test sur contrat), bordures de tableaux bloquant la sélection (test sur fiche de paie).
- **Dépendances obsolètes** : basé sur une ancienne version de Poppler, maintenance upstream quasi arrêtée, images Docker communautaires cassées ou datées.
- **Déploiement horrible** : binaire C++ à embarquer (pas de pip install), douloureux sur Docker, macOS et Windows.

"Interactif" au sens marketing (hover, zoom) — mais pas l'interactivité d'exploitation (sélection, copie de zones). Stand-by : réveil uniquement sur demande client réelle, idéalement avec un outil d'une autre famille (extraction de structure plutôt que reproduction visuelle). Trace : [_archive/pdf2htmlex-standby.md](../_archive/pdf2htmlex-standby.md).

---

## Règle de décision (rappel)

- Librairie ✅ Node.js → intégrée directement dans l'API Node.
- Librairie ❌ Python uniquement → micro-service Python séparé dans Docker (une seule fois le pattern établi — pas de fiche dédiée pour l'instant).
