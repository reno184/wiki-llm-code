# wiki-ollama — LLM local : généralités + détail complet

## Généralités : comment ça tourne sur le Mac

- **Ollama s'utilise en client desktop sur macOS** : on installe l'app, pas un serveur à configurer.
- **À partir du moment où le CLI est lancé, cela lance une tâche de fond `ollama serve`** : le serveur HTTP tourne en arrière-plan, en écoute sur `http://localhost:11434`. On ne le voit pas, mais c'est lui qui répond à tous les appels (CLI, curl, Node, Python).
- Conséquence : tant que l'app / le CLI est lancé, **l'API locale est disponible** ; si rien ne répond sur le port 11434, c'est que la tâche de fond n'est pas lancée → ouvrir l'app.
- Tout vit en local (modèles, poids, données) : **rien ne sort de la machine** → données sensibles OK (santé, contrats).

## Ligne de commande

- **CLI** : `ollama run <modele>`, `ollama pull <modele>`, `ollama list`, `ollama rm <modele>`.

  > Attention les modèles sont téléchargés sur le mac ils pèsent parfois plusieurs giga donc ne pas éhsiter à supprimer les models qui ne servent pas.

## Pattern production : Ollama chez le client (médecin / avocat)

Pattern mutualisé des deux produits (voir [../../business-idea/ia-medical-ocr-consultation.md](../../business-idea/ia-medical-ocr-consultation.md) et [../../business-idea/ia-legaltech-contrat.md](../../business-idea/ia-legaltech-contrat.md)) :

- **Workflow full Node.js, zéro Python** : API **Express** + **LangChain** pour donner les instructions au modèle (squelette : [../../use-cases/anonymisation/langchain-api.md](../../use-cases/anonymisation/langchain-api.md)).
- **Ollama installé en local chez chaque client** pour l'accès GPU (le GPU est chez le client, pas chez l'éditeur — le prérequis matériel est cadré dans l'offre).

- **Modèle** : frob/qwen3.5-instruct:9b pour la vision (voir section ci-dessous).

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

## Modèle vision de référence : frob/qwen3.5-instruct:9b

→ Détail complet (tests, périmètre de validité, paramètres) : [wiki-local-model.md](wiki-local-model.md).

## Limites constatées (leçon clé)

- **Prompt avec un document long en entier = échec** : test réel (contrat 7 pages, markdown complet) avec qwen3.5:9b — correct au début, puis paragraphes perdus et sortie tronquée après ~4 pages. Toujours découper (chunks par sections/pages), jamais tout le document en un prompt.
- Conséquence produit : le LLM local est **cantonné au court format** (1-2 pages) dans les pipelines clients ; le long format passe par PyMuPDF ciblé + human-in-the-loop (avocats).

## Règles

- Un modèle non testé n'entre pas dans cette fiche (une seule version vivante par besoin).

## Liens

- Installation sur PC client Windows : [wiki-ollama-install-windows.md](wiki-ollama-install-windows.md)
- Branchement Node (LangChain `ChatOllama`) : [wiki-ollama-langchain.md](wiki-ollama-langchain.md)
- Conteneur API → Ollama natif (compose, `host.docker.internal`) : [wiki-ollama-docker.md](wiki-ollama-docker.md)
- Use case OCR : [../../use-cases/ocr-llm/README.md](../../use-cases/ocr-llm/README.md)
- Reformatage sémantique (chunking obligatoire) : [../../use-cases/reformat-markdown/README.md](../../use-cases/reformat-markdown/README.md)
- Entrée libraries.md : Node.js / Python ✅ les deux (API HTTP locale)

## État

**vivant** — CLI desktop installé, tâche de fond `ollama serve` vérifiée ; frob/qwen3.5-instruct:9b validé pour l'OCR vision court (imprimé convenable, manuscrit parfait, déployé chez le client dans les deux produits) ; échec constaté sur reformatage de document long sans chunking (contrat 7 pages) ; pattern production (Express + LangChain + Docker/host.docker.internal) défini octobre 2026, à valider au premier déploiement.
