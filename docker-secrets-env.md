# docker Secrets

### dockerignore

- ajouter systèmatiquement .env au dockerignore sinon il sera inclus dans l'image déployée sur le serveur
- Secrets injectés uniquement au **runtime** (compose `environment`/`env_file`), jamais au build

### Todo

- Voir avec mistral pour la gestion des secrets dans le VPS Docker Secrets ou un secrets manager.

Docker Compose **Secrets** (`secrets:` → fichier monté `/run/secrets/<nom>`) : non visibles dans `docker inspect`, lisibles seulement par le process

Comprendre pourquoi il est possible de lire une image en clair avec docker inspect, comment crypter l'image

