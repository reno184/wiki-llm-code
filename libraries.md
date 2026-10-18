---
name: libraries
layer: reference
version: 4
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

## mdformat — normalisation Markdown

Reformate le Markdown de façon cohérente : espaces, wrapping, listes, tableaux. Ne touche pas aux niveaux de titres.

→ Passe 1 du use case correction-syntax-markdown.

**Node.js / Python : ❌ Python uniquement** → candidat micro-service Python, ou remplacé par **Prettier** (voir ci-dessous) si Node-only suffisant.

---

## Prettier — formatage Markdown

Formate aussi le Markdown (`npx prettier --write doc.md`) : mise en page, wrapping, tableaux. Alternative Node à mdformat. **Ne détecte pas les sauts de niveau de titres** : la détection des niveaux relève de la passe IA locale (Ollama), relue.

**Node.js / Python : ✅ Node.js** (npm).

---

## Ollama — LLM local (via llama.cpp)

Runtime de modèles locaux (`llama3.1`, `mistral`, `qwen2.5`…). API HTTP locale + CLI `ollama run`. Dans la chaîne : revue **sémantique** des niveaux de titres (un titre logiquement H2 resté H3), passe 2 optionnelle du use case correction-syntax-markdown.

⚠️ Un petit modèle local (7–8B) peut reformuler ou halluciner → sortie toujours relue, jamais en passe automatique silencieuse.

**Node.js / Python : ✅ les deux** — appel HTTP vers l'API locale Ollama depuis l'API Node, bindings Python existants.

---

## LangChain — orchestration LLM

Chaînage de prompts, loaders, modèles. Utilisé dans l'API Node (anonymisation).

**Node.js / Python : ✅ les deux** — équivalents (LangChain.js / LangChain Python). Rester côté Node OK.

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
