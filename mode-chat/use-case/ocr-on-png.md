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

## État

**vivant** — 2 tests réels validés octobre 2026 : Harry Potter (convenable, contournement découpage) et note manuscrite médicale (parfait).
