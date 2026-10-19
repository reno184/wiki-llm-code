# PyMuPDF4LLM (PDF → Markdown automatique) — ARCHIVÉ

**Archivé (oct. 2026)** : décision de rester **uniquement sur `pymupdf.get_text`** (déterministe, ciblé). PyMuPDF4LLM est écarté de toute la chaîne.

## Rôle d'origine

S'appuyait sur PyMuPDF pour analyser l'agencement visuel (titres, tableaux, listes) et reconstituer le document en Markdown automatique. Était le moteur du use case pdf-to-md (conversion intégrale).

## Leçons de l'échec (test long contrat, oct. 2026)

- **Le pipeline complet était une usine à gaz** : boucle convert-to-md (PyMuPDF4LLM) → post-traitement sémantique Ollama (Qwen) → reconsolidation des chunks — hyper long, complexe, fragile.
- **Qwen a halluciné à partir de la 4ème page** du contrat : déception totale sur le post-traitement sémantique d'un document long. Cohérent avec la leçon déjà gravée (reformat-markdown, ia-local/models) : un doc long en prompts LLM local = échec.
- Le chunking censé contourner l'hallucination a déplacé le problème (complexité de réassemblage) sans le résoudre.

## Limites constatées de l'outil lui-même (nuance importante)

- **PyMuPDF4LLM ne comprend pas sémantiquement le document** : il devine la structure à partir de l'agencement visuel (positions, tailles de police) — le rendu Markdown est **hyper perfectible** (titres ratés, hiérarchie approximative, tableaux imparfaits).
- **En revanche, il n'oublie AUCUN mot du texte** — contrairement à un LLM : zéro hallucination, zéro omission sur la matière textuelle. C'est une extraction déterministe complète, la structure seule est faible.

→ Nuance pour un futur choix d'outil : le défaut de PyMuPDF4LLM est **qualitatif** (structure), pas **quantitatif** (complétude). Un LLM est exactement l'inverse (structure plausible, complétude non garantie sur longs docs).

## Décision

**`pymupdf.get_text` uniquement** : extraction déterministe de zones ciblées (pdf-to-text, human-in-the-loop) — le LLM local ne participe plus à la transformation de documents longs.

## Réveil si

Besoin réel de Markdown structuré intégral (pas couvert par l'extraction ciblée) **ET** un moteur fiable : soit un modèle (local ou API) validé sur documents longs sans hallucination, soit une alternative déterministe. Ne pas réintroduire le trio PyMuPDF4LLM + Ollama + reconsolidation sans avoir validé le point faible (hallucination page 4+).

## État

**archivé (écarté)** — trace conservée pour ne pas retenter le même pipeline.
