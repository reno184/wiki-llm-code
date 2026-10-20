---
use-case: anonymisation
layer: backend-node
version: 0 (squelette)
---

# API Node : anonymisation (LangChain + qwen3.5)

## Contexte d'usage
API Node.js appelant qwen3.5 via LangChain avec le prompt canonique chargé au démarrage.

## Structure (à définir)

- Endpoint : `POST /anonymize` (body : document)
- Loader prompt (déjà dans prompts/anonymisation.md) :

```js
import { readFile } from "fs/promises";
import { PromptTemplate } from "@langchain/core/prompts";

const raw = await readFile("prompts/anonymisation.md", "utf-8");
const promptText = raw.split("<!-- DEBUT PROMPT -->")[1]
                     .split("<!-- FIN PROMPT -->")[0].trim();
const prompt = PromptTemplate.fromTemplate(promptText);
```

## Paramètres qwen3.5

→ Référence unique : [../../mode-chat/code-subject/wiki-local-model.md](../../mode-chat/code-subject/wiki-local-model.md) (temperature 0, top_p 1, enable_thinking false).

## Notes
- Jamais de copie du prompt dans le code : toujours chargé depuis le repo/fichier.
- À compléter avec le code réel de l'API.
