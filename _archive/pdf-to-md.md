# Use case pdf-to-md (conversion intégrale PDF → Markdown) — ARCHIVÉ

**Archivé (oct. 2026)** : son moteur était PyMuPDF4LLM, écarté après échec du pipeline complet (voir [_archive/pymupdf4llm.md](pymupdf4llm.md)).

## Rôle d'origine

Conversion intégrale d'un PDF en Markdown : sortie = brouillon pour relecture, ou entrée de l'anonymisation.

## Pourquoi archivé

- Moteur PyMuPDF4LLM écarté (le pipeline avec post-traitement Ollama = usine à gaz, hallucination Qwen dès la 4e page).
- OCRmyPDF déjà archivé avant (chaîne 100% PyMuPDF, périmètre sans scannés) → le use case n'avait plus de prétraitement ni de moteur.

## Ce qui le remplace

- **Extraction ciblée `get_text`** : [use-cases/pdf-to-text/README.md](../use-cases/pdf-to-text/README.md) — l'avocat cible ses zones, PyMuPDF sort le texte de façon déterministe, human-in-the-loop.
- Le besoin « brouillon intégral pour relecture » n'est plus couvert — assumé : la valeur produit est dans le ciblage, pas dans la conversion exhaustive.

## Réveil si

Besoin réel de conversion intégrale fiable (client exigeant un export complet du document) — condition : un moteur validé sans hallucination (voir fiche PyMuPDF4LLM archivée).

## État

**archivé (écarté)** — remplacé par pdf-to-text (get_text ciblé).
