---
use-case: anonymisation
layer: backend-python
version: 0 (squelette)
---

# Pipeline Python : anonymisation

## Contexte d'usage
Fonctions Python appelées en amont de l'API LLM : découpage du document, préparation, réassemblage de la sortie.

## Fonctions (à définir)

- `load_document(path)` — chargement du document (format, encodage)
- `chunk_document(text, max_tokens)` — découpage respectant structure/paragraphes (le prompt exige de préserver la mise en forme)
- `reassemble(chunks)` — réassemblage + cohérence des tags entre chunks ([NOM_1] identique sur tout le document)

## Notes
- Points durs anticipés : cohérence des tags numérotés à travers les chunks, mapping table globale.
- À compléter avec le code réel.
