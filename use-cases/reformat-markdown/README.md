---
use-case: reformat-markdown
version: 2
---

# Use case : reformatage sémantique Markdown (reformat-markdown)

Corrige ce que les passes déterministes (Prettier) ne peuvent pas corriger : tableaux markdown emboîtés dans des listes, hiérarchie de titres illogique, balises `**strong**` inutiles dans les titres. Reformatage **sémantique** par LLM, complément du use case [correction-syntax-markdown](../correction-syntax-markdown/README.md) (syntaxe mécanique).

## Positionnement (octobre 2026)

**Repositionné au rang labo/entraînement.** Ce use case n'est plus le pipeline cible d'un produit : les IA locales sont décevantes pour la transformation de documents longs (contrats, fiches de paie), même avec chunking — et le niveau d'exigence juridique (une clause perdue = risque) rend une passe LLM non relue inacceptable en production. Les produits clients reposent soit sur **PyMuPDF ciblé + human-in-the-loop** (avocats, cas principal), soit sur **qwen vision court format** (médecins, notes manuscrites avocats) — dans les deux cas avec relecture humaine systématique.

Le reformatage sémantique par LLM reste utile comme **passe d'assistance à la relecture** (suggestion, jamais passe silencieuse), sur des morceaux courts, sortie toujours relue.

## Test réel (octobre 2026, contrat de travail 7 pages)

Chaîne testée :
1. PyMuPDF4LLM sur le contrat → Markdown **100% fidèle** ("prédictif" : pas un mot manquant, toutes les infos présentes) mais **sémantiquement mauvais** : tableaux dans les listes, titres sans hiérarchie, strong inutile dans les titres.
2. Markdown complet envoyé à Ollama CLI desktop (qwen3.5:9b) avec prompt de reformatage → **OK au début, catastrophe après ~4 pages** : paragraphes manquants, sortie arrêtée avant la fin du document.

## Leçons (règles vivantes)

- **Jamais un document long en un seul prompt** : le modèle local dégrade puis tronque. Toujours **découper** (par sections titrées, ou par page via `page_chunks` de PyMuPDF4LLM) et reformater section par section.
- **Contrôle de complétude obligatoire** : comparer entrée/sortie (nombre de paragraphes, de titres, longueur) — un reformatage silencieusement amputé est pire que pas de reformatage.
- **Le reformatage sémantique ≠ correction syntaxique** : Prettier d'abord (mécanique), LLM ensuite seulement sur les défauts structurels, par morceaux, avec relecture humaine.
- Le LLM local reformule/hallucine → le prompt doit imposer "ne pas reformuler le texte, uniquement la structure".
- **Même chunké, ne pas bâtir un produit dessus** : le niveau d'exigence (juridique) + les limites des modèles locaux sur machines cibles rendent la transformation LLM longue non fiable. Le produit avocat assume le human-in-the-loop (PyMuPDF ciblage Vue.js) à la place.

## Chaîne cible (assistance à la relecture, jamais automatique)

```
Markdown source (sortie pdf-to-md)
   │
   ▼
Découpage (sections ou page_chunks)
   │
   ▼
LLM local par morceau (prompt : corriger structure, ne pas reformuler)
   │
   ▼
Contrôle complétude (paragraphes/titres entrée vs sortie) + relecture humaine
```

## Fiches liées

- [../correction-syntax-markdown/README.md](../correction-syntax-markdown/README.md) — passes déterministes (avant ce use case)
- [../../ia-local/ollama.md](../../ia-local/ollama.md), [../../ia-local/models.md](../../ia-local/models.md) — limites constatées du modèle local
- [../pdf-to-md/README.md](../pdf-to-md/README.md) — amont
- [../../business-idea/ia-legaltech-contrat.md](../../business-idea/ia-legaltech-contrat.md) — décision produit qui repositionne ce use case

## État

**vivant (rang labo/assistance)** — leçon validée sur échec réel (contrat 7 pages, qwen3.5:9b) ; repositionné hors pipeline produit (oct. 2026) : le long format passe par PyMuPDF ciblé + human-in-the-loop.
