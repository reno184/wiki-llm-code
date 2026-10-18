# Modèles locaux Ollama

Usage: inventaire des modèles Ollama installés et validés. Une seule version vivante par besoin (règle du wiki).

## Modèle vision de référence : frob/qwen3.5-instruct:9b

- **Besoin couvert** : OCR / extraction d'informations depuis une image (PNG capture d'écran, scan, photo).
- **Testé** : OCR d'une fiche de lecture Harry Potter (PNG) → résultat convenable ; fausse note manuscrite médicale (PNG généré par IA) → **parfait, réussite absolue**. Voir [use-cases/ocr-llm/README.md](../use-cases/ocr-llm/README.md).
- **Taille** : ~9B paramètres — tourne sur machine locale via CLI desktop Ollama.

## Test reformatage Markdown long : échec (contrat 7 pages, octobre 2026)

- **Besoin visé** : reformatage sémantique du markdown complet d'un contrat (tableaux dans listes, hiérarchie titres, strong inutiles).
- **Résultat** : correct au début, **catastrophe après ~4 pages** — paragraphes manquants, sortie arrêtée avant la fin.
- **Verdict** : qwen3.5:9b validé pour vision/extraction courte, **PAS pour le reformatage de documents longs en un seul prompt**. Contournement : chunking par sections (à retester), voir [use-cases/reformat-markdown/README.md](../use-cases/reformat-markdown/README.md).

## Règles

- Documenter ici chaque modèle réellement installé + le besoin qu'il couvre.
- Un modèle non testé n'entre pas dans cette fiche.

## État

**vivant** — frob/qwen3.5-instruct:9b validé pour l'OCR vision (imprimé convenable, manuscrit parfait) ; échec constaté sur reformatage de document long sans chunking (contrat 7 pages).
