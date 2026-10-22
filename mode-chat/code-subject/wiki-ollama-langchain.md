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

## Envoyer une image (multimodal) — base64 Data URL

Un buffer binaire ne passe pas dans un payload JSON : on l'encode en base64 et on l'inline dans le message via un Data URL.

```js
// fileData.buffer = Buffer issu de l'upload (busboy), fileData.mimeType ex. "image/png"
const formattedImage = `data:${fileData.mimeType};base64,${fileData.buffer.toString("base64")}`;

const message = new HumanMessage({
    content: [
        { type: "text", text: promptText },
        { type: "image_url", image_url: { url: formattedImage } }
    ]
});

const response = await llm.invoke([message]);
```

**Pourquoi le base64 :**
- JSON ne transporte pas d'octets bruts → le base64 transforme le binaire en texte ASCII sûr pour la requête HTTP/JSON.
- Le Data URL `data:<mimeType>;base64,<...>` indique au LLM le type d'image + les données en une seule chaîne (il décode et retrouve le binaire).
- Alternative : héberger l'image (S3…) et envoyer une URL publique — le base64 évite l'externalisation de la donnée (plus simple + confidentiel).

**Piège :** le base64 augmente la taille de ~33 %. OK pour screenshots ; busboy ne passe pas par la limite bodyParser, donc pas de blocage Express.

## Pièges à ne pas oublier

- **`baseUrl` selon le contexte** :
  - Depuis l'hôte (API Node locale) : `http://localhost:11434`
  - **Depuis Docker** : `localhost` désigne le conteneur, pas la machine → utiliser `http://host.docker.internal:11434`

## État

**vivant** — pattern de branchement validé. Section image/base64 ajoutée le 22/10/2026 (route `/screenshotOCR`, image uploadée → Gemini via LangChain).
