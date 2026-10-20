# wiki-ollama — LLM local : généralités

Usage: connaissances générales mode chat Renaud (jamais chargé par un agent). Fiche d'entrée sur Ollama ; le détail technique vit dans [../../ia-local/ollama.md](../../ia-local/ollama.md) et les modèles dans [../../ia-local/models.md](../../ia-local/models.md).

## Généralités : comment ça tourne sur le Mac

- **Ollama s'utilise en client desktop sur macOS** : on installe l'app, pas un serveur à configurer.
- **À partir du moment où le CLI est lancé, cela lance une tâche de fond `ollama serve`** : le serveur HTTP tourne en arrière-plan, en écoute sur `http://localhost:11434`. On ne le voit pas, mais c'est lui qui répond à tous les appels (CLI, curl, Node, Python).
- Conséquence : tant que l'app / le CLI est lancé, **l'API locale est disponible** ; si rien ne répond sur le port 11434, c'est que la tâche de fond n'est pas lancée → ouvrir l'app.
- Tout vit en local (modèles, poids, données) : **rien ne sort de la machine** → données sensibles OK (santé, contrats).

## Ce qu'il faut retenir en bref

- Appel type : `curl http://localhost:11434/api/generate` ; CLI : `ollama run / pull / list / rm`.
- Vision : champ `images` en base64 dans l'appel API.
- Modèle vision de référence : **frob/qwen3.5-instruct:9b** (OCR/extraction courte validé).
- Piège clé : **jamais un document long en un seul prompt** (échec constaté contrat 7 pages) → chunking obligatoire.
- Piège Docker : depuis un conteneur, `localhost` ≠ hôte → `host.docker.internal`.

## Liens vers les fiches détaillées

- Runtime, setup, patterns production : [../../ia-local/ollama.md](../../ia-local/ollama.md)
- Inventaire des modèles validés : [../../ia-local/models.md](../../ia-local/models.md)

## État

**vivant** — setup desktop macOS réel, serveur d'arrière-plan vérifié.
