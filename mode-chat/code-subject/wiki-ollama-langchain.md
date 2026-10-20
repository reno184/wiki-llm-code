# wiki-ollama-langchain — appeler Ollama depuis Node (LangChain)

Usage: snippet de branchement API Node → Ollama. Mode chat Renaud (jamais chargé par un agent). Runtime général → [wiki-ollama.md](wiki-ollama.md), compose conteneur → Ollama natif → [wiki-ollama-docker.md](wiki-ollama-docker.md).

## Import et instanciation

```js
import { ChatOllama } from "@langchain/ollama";

export async function testLlama() {
    const llm = new ChatOllama({
        baseUrl: "http://ollama:11434",
        model: "llama3.2:latest",
        temperature: 0.3
    });
```

## Pièges à ne pas oublier

- **`baseUrl` selon le contexte** :
  - Depuis l'hôte (API Node locale) : `http://localhost:11434`
  - **Depuis Docker** : `localhost` désigne le conteneur, pas la machine → utiliser `http://host.docker.internal:11434`
  - **Depuis un compose où Ollama tourne en service** : `http://ollama:11434` (nom du service compose, comme dans le snippet)
- **Paramètres de production** : temperature `0`, top_p `1`, `enable_thinking: false` si modèle think → référence : [wiki-local-model.md](wiki-local-model.md) (le `temperature: 0.3` du snippet est un exemple, la norme production est 0).
- Le package : `npm install @langchain/ollama langchain`.
- API complète (prompt canonique chargé au démarrage, squelette Express) : [../../use-cases/anonymisation/langchain-api.md](../../use-cases/anonymisation/langchain-api.md)

## État

**vivant** — pattern de branchement validé.
