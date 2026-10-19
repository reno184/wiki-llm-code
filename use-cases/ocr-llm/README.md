# Use case : OCR via LLM vision local (Ollama)

Entrée : image (PNG capture, scan). Sortie : Markdown ou informations extraites. Alternative aux outils OCR Python classiques quand l'image est complexe ou que le texte n'est pas filaire.

## Test de référence (octobre 2026)

- **Outil** : CLI desktop Ollama, modèle vision frob/qwen3.5-instruct:9b.
- **Entrée** : fiche de lecture Harry Potter, fichier PNG.
- **Prompt exact** : « transforme en un markdown, sans extra, sans commentaires ».
- **Résultat** : convenable, SAUF la liste de personnages en fin de fiche (interligne très serré) → 3 personnages ratés (les derniers).
- **Contournement validé** : capture ciblée sur la zone problématique uniquement (la liste des personnages) → résultat nickel.

## Test 2 : note manuscrite médicale (octobre 2026)

- **Outil** : CLI desktop Ollama, modèle vision frob/qwen3.5-instruct:9b.
- **Entrée** : fausse note manuscrite médicale (générée par IA, PNG stocké dans le dossier « ressources » du Finder).
- **Résultat** : **parfait — réussite absolue.** Zéro zone ratée, manuscrit lu intégralement.

→ Confirme la hiérarchie ci-dessous : sur du manuscrit, les outils Python classiques rendent nul, le LLM vision local réussit.

### Leçon

Un LLM vision peut rater une zone dense d'un document long. **Le découpage de l'image sur les zones denses résout le problème.** Pour un document à zones hétérogènes : découper et passer les zones séparément.

## Comparatif : LLM vision vs outils OCR Python

Sur une image très compliquée (mise en page dense, note manuscrite), les outils Python classiques rendent un résultat nul :

- **PyMuPDF / PyMuPDF4LLM** : conçus pour le texte filaire des PDF, pas la compréhension d'une image complexe. → nul sur scan dense ou manuscrit.
- **OCRmyPDF / Tesseract** : reconnaissance caractère par caractère, échoue sur manuscrit et mise en page serrée.

→ **Hiérarchie de décision (version vivante)** — le critère décisif est la **longueur/densité** du document :

1. PDF texte natif → PyMuPDF4LLM (chaîne pdf-to-md) ou PyMuPDF ciblé (extraction de zones, fiche infra/pymupdf.md).
2. Scan propre / texte imprimé → OCRmyPDF puis PyMuPDF4LLM.
3. **Image complexe, scan dense ou manuscrit → LLM vision local (Ollama / qwen)** — sans hésiter si le document est **court** (1-2 pages) : c'est la zone de validité prouvée du modèle (2 tests réussis).
4. **Document long/dense multi-pages → pas de LLM vision en pipeline automatique** : échec constaté au-delà de ~4 pages (leçon contrat). Passer par PyMuPDF ciblé + human-in-the-loop.

**Le critère est la longueur/densité, pas « IA locale oui/non »** : le LLM vision local est validé et rentable pour l'extraction courte, disqualifié pour la transformation de documents longs.

## Rappel futur

**Dans 1 an, si je cherche un bon outil d'OCR (ex. note manuscrite ou document court) : la réponse est ollama/qwen vision local, pas un outil Python classique.** Cette fiche est le point d'entrée.

## Idées business associées

- [business-idea/ia-medical-ocr-consultation.md](../../business-idea/ia-medical-ocr-consultation.md) : extraction de bilans/notes dactylographiées 1-2 pages via IA locale — **le LLM vision y est le moteur principal** (full Node.js : Express + LangChain + Ollama chez le client).
- [business-idea/ia-legaltech-contrat.md](../../business-idea/ia-legaltech-contrat.md) : le LLM vision n'y sert que pour les notes manuscrites ponctuelles (le cas principal = PyMuPDF ciblé + human-in-the-loop).

## État

**vivant** — 2 tests réels validés octobre 2026 : Harry Potter (convenable, contournement découpage) et note manuscrite médicale (parfait). Périmètre de validité précisé : court = validé, long = échec.
