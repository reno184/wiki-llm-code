# Tesseract OCR & Docling — écartés de la chaîne d'extraction (archivés)

Stack: Tesseract (OCR moteur bas niveau), Docling (conversion document → structure). Évalués comme solutions autonomes pour l'extraction de documents d'entreprise (contrats, fiches de paie).
Usage: trace de décision — solutions testées/évaluées, non retenues, écartées de la chaîne.

## Décision

**Écartés.** Ne pas réutiliser Tesseract ni Docling comme solutions autonomes d'extraction. La chaîne retenue pour l'analyse de contrats et de fiches de paie est **PyMuPDF** (extraction ciblée, fiche infra/pymupdf.md) avec **ciblage humain via interface Vue.js** : l'avocat sélectionne les zones dans l'interface, PyMuPDF transforme en texte (clip + search_for + find_tables), human-in-the-loop systématique — pas de process 100% automatique visé.

## Pourquoi

- **Résultats décevants** sur les documents cibles (contrats, fiches de paie) : qualité insuffisante par rapport au duo PyMuPDF + ciblage humain.
- **Docling** : solution de conversion automatique structurée, même famille que les approches "tout automatique" déjà écartées — la fiabilité vient du ciblage humain sur documents denses, pas de la devinette automatique.
- **Tesseract en direct** : OCR bas niveau, déjà en retrait derrière ocrmypdf (qui l'utilise en interne et apporte le prétraitement). L'utiliser nu n'apporte rien de plus que la chaîne OCRmyPDF existante pour les scannés.
- Cohérent avec l'architecture produits : extraction déterministe PyMuPDF pour les documents longs structurés (avocats/M&A), LLM vision local (Ollama/qwen) pour le court dactylographié/manuscrit (médecins, notes de réunion) — aucune place pour un OCR automatique généraliste.

## Nuance à conserver

Tesseract reste **sous le capot d'OCRmyPDF** (fiche libraries.md / use-cases/pdf-to-md) : le moteur n'est pas banni, c'est son **usage autonome** qui est archivé. La chaîne scannés → OCRmyPDF → PyMuPDF4LLM reste vivante pour le labo.

## Anti-patterns

- Retenter Tesseract ou Docling "pour voir si ça marche mieux" sur un nouveau type de document sans demande client réelle.
- Choisir un convertisseur automatique généraliste là où un ciblage humain + PyMuPDF fait mieux, plus simple et vérifiable.

## Conditions de réveil

- Une demande client réelle sur un type de document que ni PyMuPDF (PDF natif), ni OCRmyPDF (scannés), ni le LLM vision (court/manuscrit) ne couvrent.
- À ce moment-là : évaluer d'abord les limites de la solution retenue (PyMuPDF + ciblage) avant d'introduire une brique de plus.

## État

**archivé (écarté)** — décision 2026-10 : résultats décevants, chaîne retenue = PyMuPDF + ciblage Vue.js + human-in-the-loop. Hors INDEX (politique _archive/), cette fiche sert uniquement de trace.
