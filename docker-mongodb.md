---
title: MongoDB dans Docker avec authentification
tags: [docker, backend, howto]
fiches-liees: [wiki-mongoose, mongodb, docker-commandes, reseau-interne-docker, docker-volumes-persistence]
---
# MongoDB dans Docker avec auth

## Le piège de base

Sans authentification, le serveur mongod **accepte toutes les connexions anonymes** : n'importe quel conteneur du réseau Docker (donc n'importe quel service compromis) peut lire et écrire dans la base. Dans le conteneur officiel, mongod écoute sur toutes les interfaces, donc il n'y a aucune protection réseau par défaut.

## Activer l'auth correctement (image officielle)

Avec l'image `mongo`, on ne passe **pas** `--auth` à la main : les variables d'environnement créent l'utilisateur root **et activent l'auth automatiquement**.

```yaml
services:
  mongodb:
    image: mongo:7
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: motdepasse  # via .env, jamais en dur
    volumes:
      - mongo_data:/data/db
    # pas de "ports:" vers l'hôte si seul le backend y accède
volumes:
  mongo_data:
```

## Se connecter

App (Mongoose) :

```js
mongodb://root:motdepasse@mongodb:27021/nombase?authSource=admin
```

Port `27017` dans le conteneur (pas 27021). `authSource=admin` obligatoire : l'utilisateur root est créé dans la base `admin`, pas dans la base applicative.

Terminal :

```bash
docker compose exec mongodb mongosh -u root -p --authenticationDatabase admin
```

## Pièges

- **Volume déjà initialisé** : `MONGO_INITDB_ROOT_USERNAME/PASSWORD` ne sont lus que si `/data/db` est **vide au premier démarrage**. Ajouter les variables sur un volume existant ne fait rien, l'auth reste désactivée. Soit repartir d'un volume vierge, soit créer l'utilisateur à la main dans mongosh.
- **Ne pas exposer le port** : si seul le backend (même réseau Docker interne) accède à Mongo, pas de mapping `ports:` vers l'hôte. Voir la fiche reseau-interne-docker.
- **authSource oublié** : erreur classique `Authentication failed` alors que le user/password sont corrects, parce que le client cherche l'utilisateur dans la base applicative au lieu de `admin`.
