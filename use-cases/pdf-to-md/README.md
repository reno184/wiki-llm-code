---
use-case: pdf-to-md
version: 4
---

# Use case : PDF → Markdown (pdf-to-md)

Convertit des PDF (scannés ou natifs) en Markdown. **Deux finalités distinctes (oct. 2026)** :

1. **Labo / back-office** : produire un **brouillon Markdown** pour relecture humaine — plus une promesse de livrable 100% automatique.
2. **Chaîne anonymisation** : la sortie reste l'entrée **exclusive** du use case [anonymisation](../anonymisation/README.md).

## Chaîne de traitement

```
PDF source
   │
   ▼
ocrmypdf                  ← 1. prétraitement (voir ocrmypdf.md : les 2 rôles)
   │
   ▼
PyMuPDF4LLM (pymupdf4llm) ← 2. conversion → Markdown
   │
   ▼
Markdown structuré → relecture humaine (brouillon) OU use-cases/anonymisation/
```

## Pourquoi OCRmyPDF systématiquement avant PyMuPDF4LLM

OCRmyPDF joue **2 rôles** dans la chaîne :

1. **Ajout d'un layer texte invisible** sur les PDF scannés (image seule) : le texte devient sélectionnable, donc extractible par PyMuPDF4LLM. Sans ce layer, PyMuPDF4LLM ne sort rien d'exploitable sur un scan.
2. **Optimisation/nettoyage du PDF** avant conversion : meilleure qualité d'extraction en sortie.

→ Toujours passer par OCRmyPDF, même si le PDF paraît « textuel » : le coût est faible et ça garantit un layer texte propre pour PyMuPDF4LLM.

## Extraction ciblée (alternative complémentaire)

Pour l'exploitation de contrats/fiches de paie (produit avocat), la conversion intégrale n'est **pas** la seule voie : **PyMuPDF en extraction ciblée** (zones sélectionnées dans une interface Vue.js, clip + search_for + find_tables) + human-in-the-loop est la chaîne retenue — voir [infra/pymupdf.md](../infra/pymupdf.md) et [business-idea/ia-legaltech-contrat.md](../business-idea/ia-legaltech-contrat.md). La chaîne ci-dessus (conversion intégrale) produit alors un **brouillon** à relire, pas un livrable automatique.

## Fiches liées
- [ocrmypdf.md](ocrmypdf.md) — commandes OCRmyPDF (2 use cases : layer texte invisible + optimisation)
- [pymupdf4llm.md](pymupdf4llm.md) — conversion Markdown via PyMuPDF4LLM
- [../infra/pymupdf.md](../infra/pymupdf.md) — extraction ciblée PyMuPDF (alternative pour l'exploitation de zones)

## Règle de synchronisation
La sortie de ce pipeline (Markdown) est l'entrée **exclusive** du use case anonymisation : format d'entrée = `text/markdown`, pas de PDF direct. La passe de contrôle qualité (correction-syntax-markdown) est archivée — relecture humaine directe du brouillon (trace : [../_archive/correction-syntax-markdown-standby.md](../_archive/correction-syntax-markdown-standby.md)).
