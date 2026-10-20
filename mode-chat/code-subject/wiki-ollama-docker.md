# wiki-ollama-docker — appeler Ollama natif depuis un conteneur (compose)

Usage: le pattern du déploiement client — code API dans Docker (protégé), Ollama **jamais dockerisé** (natif, GPU sur l'hôte). Le conteneur appelle l'Ollama de la machine. Mode chat Renaud, jamais chargé par un agent. Runtime général → [wiki-ollama.md](wiki-ollama.md), installation Ollama Windows → [wiki-ollama-install-windows.md](wiki-ollama-install-windows.md), branchement LangChain → [wiki-ollama-langchain.md](wiki-ollama-langchain.md).

## docker-compose.yml

```yaml
services:
  app:
    build: .
    working_dir: /app
    volumes:
      - .:/app
    tty: true
    stdin_open: true
    environment:
      - OLLAMA_BASE_URL=http://host.docker.internal:11434
```

## Clé du pattern

- **`host.docker.internal`** : depuis le conteneur, `localhost` désigne le conteneur, pas la machine → l'Ollama natif de l'hôte s'atteint via `host.docker.internal:11434`.
- **Ollama n'est PAS dans le compose** : pas de service ollama, pas de bloc GPU, pas de conflit de port 11434. Ollama est installé en natif sur le PC client.
- **URL passée par env** (`OLLAMA_BASE_URL`) : le code lit `process.env.OLLAMA_BASE_URL` (jamais en dur), donc le même conteneur marche en local (`http://localhost:11434`) et chez le client sans rebuild — seul le compose change.
- Côté LangChain : `baseUrl: process.env.OLLAMA_BASE_URL` (cf. [wiki-ollama-langchain.md](wiki-ollama-langchain.md)).

## Checklist

- [ ] Ollama installé en natif et lancé sur l'hôte (port 11434 répond)
- [ ] `OLLAMA_BASE_URL=http://host.docker.internal:11434` dans le compose
- [ ] Le code lit l'URL depuis l'env, jamais en dur
- [ ] Test : appel API depuis le conteneur → réponse du modèle

## État

**vivant** — pattern de déploiement client (code dockerisé + Ollama natif GPU).
