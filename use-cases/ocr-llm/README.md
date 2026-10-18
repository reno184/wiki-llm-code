# Use case : OCR via LLM vision local (Ollama)

Entrée : image (PNG capture, scan). Sortie : Markdown ou informations extraites. Alternative aux outils OCR Python classiques quand l'image est complexe ou que le texte n'est pas filaire.

## Test de référence (octobre 2026)

- **Outil** : CLI desktop Ollama, modèle vision frob/qwen3.5-instruct:9b.
- **Entrée** : fiche de lecture Harry Potter, fichier PNG.
- **Prompt exact** : « transforme en un markdown, sans extra, sans commentaires ».
- **Résultat** : convenable, SAUF la liste de personnages en fin de fiche (interligne très serré) → 3 personnages ratés (les derniers).
- **Contournement validé** : capture ciblée sur la zone problématique uniquement (la liste des personnages) → résultat nickel.

### Leçon

Un LLM vision peut rater une zone dense d'un document long. **Le découpage de l'image sur les zones denses résout le problème.** Pour un document à zones hétérogènes : découper et passer les zones séparément.

## Comparatif : LLM vision vs outils OCR Python

Sur une image très compliquée (mise en page dense, note manuscrite), les outils Python classiques rendent un résultat nul :

- **PyMuPDF / PyMuPDF4LLM** : conçus pour le texte filaire des PDF, pas la compréhension d'une image complexe. → nul sur scan dense ou manuscrit.
- **OCRmyPDF / Tesseract** : reconnaissance caractère par caractère, échoue sur manuscrit et mise en page serrée.

→ **Hiérarchie de décision (version vivante)** :

1. PDF texte natif → PyMuPDF4LLM (chaîne pdf-to-md).
2. Scan propre / texte imprimé → OCRmyPDF puis PyMuPDF4LLM.
3. **Image complexe, scan dense ou manuscrit → LLM vision local (Ollama / qwen)**.

## Rappel futur

**Dans 1 an, si je cherche un bon outil d'OCR (ex. note manuscrite) : la réponse est ollama/qwen vision local, pas un outil Python classique.** Cette fiche est le point d'entrée.

## Idée business associée

→ [business-idea/ia-medical-ocr-consultation.md](../../business-idea/ia-medical-ocr-consultation.md) : extraction d'infos depuis compte rendu médical scanné via IA locale.

## État

**vivant** — test réel validé octobre 2026 sur Harry Potter, contournement découpage validé.
