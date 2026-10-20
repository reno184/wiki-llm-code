# wiki-ollama-langchain — appeler Ollama depuis Node (LangChain)

Usage: snippet de branchement API Node → Ollama. Mode chat Renaud (jamais chargé par un agent). Runtime général → [wiki-ollama.md](wiki-ollama.md), compose conteneur → Ollama natif → [wiki-ollama-docker.md](wiki-ollama-docker.md).

## Import et instanciation

```js
import { ChatOllama } from "@langchain/ollama";

const llm = new ChatOllama({
    baseUrl: process.env.OLLAMA_BASE_URL || "http://localhost:11434",
    model: "frob/qwen3.5-instruct:9b",
    temperature: 0.3
})

const response = await llm.invoke("Dis moi bonjour")

res.json({message: response.content})
```

## Pièges à ne pas oublier

- **`baseUrl` selon le contexte** :
  - Depuis l'hôte (API Node locale) : `http://localhost:11434`
  - **Depuis Docker** : `localhost` désigne le conteneur, pas la machine → utiliser `http://host.docker.internal:11434`

## État

**vivant** — pattern de branchement validé.
