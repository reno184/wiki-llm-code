# Ollama — runtime de LLM local

Usage: exécuter des LLM en local sans cloud — OCR vision, revue sémantique, extraction depuis documents. Deux modes : labo (CLI desktop macOS) et **production = déployé chez le client** (médecins, avocats).

## Setup labo (version vivante)

- **Installation** : CLI desktop Ollama (app macOS). L'app lance le serveur local automatiquement.
- **API HTTP locale** : `http://localhost:11434` — appelable depuis Node, Python, curl.
- **CLI** : `ollama run <modele>`, `ollama pull <modele>`, `ollama list`, `ollama rm <modele>`.
- Les modèles et leurs poids vivent en local, rien ne sort de la machine → données sensibles OK (santé, contrats).

## Pattern production : Ollama chez le client (médecin / avocat)

Pattern mutualisé des deux produits (voir [business-idea/ia-medical-ocr-consultation.md](../business-idea/ia-medical-ocr-consultation.md) et [business-idea/ia-legaltech-contrat.md](../business-idea/ia-legaltech-contrat.md)) :

- **Workflow full Node.js, zéro Python** : API **Express** + **LangChain** pour donner les instructions au modèle (squelette : [use-cases/anonymisation/langchain-api.md](../use-cases/anonymisation/langchain-api.md)).
- **Ollama installé en local chez chaque client** pour l'accès GPU (le GPU est chez le client, pas chez l'éditeur — le prérequis matériel est cadré dans l'offre).
- **Modèle** : frob/qwen3.5-instruct:9b pour la vision (voir [models.md](models.md)).
- **Accès depuis conteneur Docker via localhost** : piège Docker — `localhost` dans un conteneur ≠ localhost de l'hôte → utiliser `host.docker.internal` (à valider au premier déploiement réel).
- Argument de vente dérivé : le LLM tourne chez le client → l'éditeur n'a même pas accès aux données (secret professionnel avocat, données de santé).

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

## Limites constatées (leçon clé)

- **Prompt avec un document long en entier = échec** : test réel (contrat 7 pages, markdown complet) avec qwen3.5:9b — correct au début, puis paragraphes perdus et sortie tronquée après ~4 pages. Toujours découper (chunks par sections/pages), jamais tout le document en un prompt. Voir [use-cases/reformat-markdown/README.md](../use-cases/reformat-markdown/README.md).
- Conséquence produit : le LLM local est **cantonné au court format** (1-2 pages) dans les pipelines clients ; le long format passe par PyMuPDF ciblé + human-in-the-loop (avocats).

## Liens

- Use case OCR : [use-cases/ocr-llm/README.md](../use-cases/ocr-llm/README.md)
- Revue sémantique Markdown : [use-cases/correction-syntax-markdown/README.md](../use-cases/correction-syntax-markdown/README.md)
- Reformatage sémantique (chunking obligatoire) : [use-cases/reformat-markdown/README.md](../use-cases/reformat-markdown/README.md)
- Entrée libraries.md : Node.js / Python ✅ les deux (API HTTP locale)

## État

**vivant** — CLI desktop installé, modèle vision testé sur OCR PNG (octobre 2026) ; pattern production chez le client (Express + LangChain + Docker/host.docker.internal) défini octobre 2026, à valider au premier déploiement.
