---
use-case: anonymisation
version: 1
---

# Use case : anonymisation de documents Markdown

Workflow complet : Markdown (produit par [pdf-to-md](../pdf-to-md/README.md)) → pipeline Python → prompt canonique → qwen3.5 → document anonymisé.

## Architecture

```
Markdown source (issu de use-cases/pdf-to-md/)
   │
   ▼
pipeline-python.md          ← chunking + préparation
   │
   ▼
prompts/anonymisation.md   ← prompt canonique (source de vérité, NE PAS dupliquer ici)
   │
   ▼
langchain-api.md            ← API Node + LangChain + qwen3.5
   │
   ▼
Document anonymisé + mapping table
```

## Format d'entrée

**Markdown uniquement** (`text/markdown`). La conversion PDF → Markdown est gérée par le use case [pdf-to-md](../pdf-to-md/README.md) (OCRmyPDF → PyMuPDF4LLM) — jamais de PDF direct ici.

## Fiches liées
- [../../prompts/anonymisation.md](../../prompts/anonymisation.md) — prompt canonique (frontmatter versionné)
- [pipeline-python.md](pipeline-python.md) — fonctions Python (chunking, réassemblage)
- [langchain-api.md](langchain-api.md) — API Node LangChain
- [../pdf-to-md/README.md](../pdf-to-md/README.md) — use case amont : PDF → Markdown

## Règle de synchronisation

Toute modification du prompt = bump `version` dans `prompts/anonymisation.md` + vérifier que le loader API charge bien le fichier (jamais de copie hardcodée) + re-test manuel dans Ollama.

Les variantes d'exécution (Ollama test / LangChain prod) sont générées à la demande via le skill Vibe `optimiser-prompt-llm` — jamais stockées ici.
