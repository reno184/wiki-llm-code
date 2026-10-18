---
name: libraries
layer: reference
version: 3
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

## markdownlint — lint et correction Markdown

Détecte et corrige les problèmes structurels d'un Markdown : MD001 (sauts de niveau de titres), MD025 (H1 multiples), MD026 (ponctuation en fin de titre), longueur de lignes, listes incohérentes… `--fix` corrige automatiquement ce qui est mécanique.

→ Passe 1 du use case [correction-syntax-markdown](use-cases/correction-syntax-markdown/README.md). **Déterministe** : jamais de reformulation du texte, reproductible.

**Node.js / Python : ✅ Node.js principalement** (markdownlint-cli2 / markdownlint-cli en npm). Bindings Python existent (markdownlint) mais l'écosystème est npm → intégrer dans l'API Node.

---

## mdformat — normalisation Markdown

Reformate le Markdown de façon cohérente : espaces, wrapping, listes, tableaux. Ne touche pas aux niveaux de titres (c'est le rôle de markdownlint).

→ Passe 2 (optionnelle) du use case correction-syntax-markdown.

**Node.js / Python : ❌ Python uniquement** → candidat micro-service Python, ou remplacé par **Prettier** (voir ci-dessous) si Node-only suffisant.

---

## Prettier — formatage Markdown

Formate aussi le Markdown (`npx prettier --write doc.md`) : mise en page, wrapping, tableaux. Alternative Node à mdformat. **Ne détecte pas les sauts de niveau de titres** : complément de markdownlint, pas un remplaçant.

**Node.js / Python : ✅ Node.js** (npm). Côté Node, préférer markdownlint + Prettier plutôt que mdformat.

---

## Ollama — LLM local (via llama.cpp)

Runtime de modèles locaux (`llama3.1`, `mistral`, `qwen2.5`…). API HTTP locale + CLI `ollama run`. Dans la chaîne : revue **sémantique** des niveaux de titres (un titre logiquement H2 resté H3), passe 3 optionnelle du use case correction-syntax-markdown.

⚠️ Un petit modèle local (7–8B) peut reformuler ou halluciner → sortie toujours relue, jamais en passe automatique silencieuse.

**Node.js / Python : ✅ les deux** — appel HTTP vers l'API locale Ollama depuis l'API Node, bindings Python existants.

---

## LangChain — orchestration LLM

Chaînage de prompts, loaders, modèles. Utilisé dans l'API Node (anonymisation).

**Node.js / Python : ✅ les deux** — équivalents (LangChain.js / LangChain Python). Rester côté Node OK.

---

## pdf2htmlEX — PDF → HTML (archivé)

Reproduit un PDF en HTML avec des blocs en position absolue, fidélité visuelle ~95 %.

⚠️ **Archivé pour problèmes de rendu** : lignes chevauchées incurables par paramétrage, bordures de tableaux bloquant la sélection. Trace de décision et conditions de réveil : [_archive/pdf2htmlex-standby.md](../_archive/pdf2htmlex-standby.md).

**Node.js / Python : ❌ outil C++ autonome** (CLI/binaire), pas une librairie Python à proprement parler.

---

## Règle de décision (rappel)

- Librairie ✅ Node.js → intégrée directement dans l'API Node.
- Librairie ❌ Python uniquement → micro-service Python séparé dans Docker (une seule fois le pattern établi — pas de fiche dédiée pour l'instant).
