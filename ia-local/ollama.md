# Ollama — runtime de LLM local

Usage: exécuter des LLM en local (macOS) sans cloud — OCR vision, revue sémantique, extraction depuis documents. Entrée = CLI desktop Ollama.

## Setup (version vivante)

- **Installation** : CLI desktop Ollama (app macOS). L'app lance le serveur local automatiquement.
- **API HTTP locale** : `http://localhost:11434` — appelable depuis Node, Python, curl.
- **CLI** : `ollama run <modele>`, `ollama pull <modele>`, `ollama list`, `ollama rm <modele>`.
- Les modèles et leurs poids vivent en local, rien ne sort de la machine → données sensibles OK (santé, contrats).

## Appel type (API generate)

```bash
curl http://localhost:11434/api/generate -d '{
  "model": "frob/qwen3.5-instruct:9b",
  "prompt": "...",
  "stream": false
}'
```

Pour la vision : champ `images` avec le contenu du fichier en base64.

## Modèles utilisés

→ Voir [models.md](models.md) — modèle vision de référence : frob/qwen3.5-instruct:9b.

## Liens

- Use case OCR : [use-cases/ocr-llm/README.md](../use-cases/ocr-llm/README.md)
- Revue sémantique Markdown : [use-cases/correction-syntax-markdown/README.md](../use-cases/correction-syntax-markdown/README.md)
- Entrée libraries.md : Node.js / Python ✅ les deux (API HTTP locale)

## État

**vivant** — CLI desktop installé, modèle vision testé sur OCR PNG (octobre 2026).
