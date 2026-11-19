# docker Secrets

### dockerignore

- ajouter systèmatiquement .env au dockerignore sinon il sera inclus dans l'image déployée sur le serveur
- Secrets injectés uniquement au **runtime** (compose `environment`/`env_file`), jamais au build

### Todo

- Voir avec mistral pour la gestion des secrets dans le VPS Docker Secrets ou un secrets manager.

Docker Compose **Secrets** (`secrets:` → fichier monté `/run/secrets/<nom>`) : non visibles dans `docker inspect`, lisibles seulement par le process

Comprendre pourquoi il est possible de lire une image en clair avec docker inspect, comment crypter l'image


### docker réseu vitruel


│ Contexte                                                                  │ MONGODB_URI                                                       │ Connexion                                                                                                                                    │
│ Dev local (node / npm run dev)                                            │ non définie                                                       │ mongodb://localhost:27017/myDb → ton Mongo local │
│docker compose up demo_backend  │ mongodb://mongo:27017/myDb │ via le réseau virtuel Docker, invisible depuis l'extérieur                                                                                   │

Tu peux empiler autant de stacks que tu veux sur le VPS : chacune a son mongo interne, aucun port 27017 n'est jamais bindé sur le host. En dev, si tu veux cibler un Mongo Docker spécifique, tu exportes juste la variable toi-même : MONGODB_URI=mongodb://localhost:27018/myDb npm run dev (avec ce Mongo-là publié sur 27018).

