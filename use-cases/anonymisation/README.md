---
use-case: anonymisation
version: 0 (squelette)
---

# Use case : anonymisation de documents

Workflow complet : document entrant → pipeline Python → prompt canonique → qwen3.5 → document anonymisé.

## Architecture

```
Document (PDF/TXT)
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

## Fiches liées
- [../../prompts/anonymisation.md](../../prompts/anonymisation.md) — prompt canonique (frontmatter versionné)
- [pipeline-python.md](pipeline-python.md) — fonctions Python (chunking, réassemblage)
- [langchain-api.md](langchain-api.md) — API Node LangChain

## Règle de synchronisation

Toute modification du prompt = bump `version` dans `prompts/anonymisation.md` + vérifier que le loader API charge bien le fichier (jamais de copie hardcodée) + re-test manuel dans Ollama.

Les variantes d'exécution (Ollama test / LangChain prod) sont générées à la demande via le skill Vibe `optimiser-prompt-llm` — jamais stockées ici.
