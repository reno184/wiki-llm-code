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

Terminal. `docker compose exec` lance mongosh dans le **conteneur actif** (déjà démarré) :

```bash
# sans auth (si l'auth est active : connexion acceptée mais toutes les commandes rejetées)
docker compose exec mongodb mongosh

# avec auth : la forme à utiliser
docker compose exec mongodb mongosh -u root -p --authenticationDatabase admin
```

Points à savoir sur `docker compose exec` :

- Pas besoin de `-it` : contrairement à `docker exec`, `compose exec` alloue un TTY par défaut dans un terminal.
- `mongodb` est le **nom du service compose**, pas le nom du conteneur : pas de piège de préfixe projet (`monprojet-mongodb-1`).
- `exec` ne marche que sur un conteneur **en cours d'exécution**. Pour un conteneur arrêté : `docker compose run --rm mongodb mongosh ...`.

## Pièges

- **Volume déjà initialisé** : `MONGO_INITDB_ROOT_USERNAME/PASSWORD` ne sont lus que si `/data/db` est **vide au premier démarrage**. Ajouter les variables sur un volume existant ne fait rien, l'auth reste désactivée. Soit repartir d'un volume vierge, soit créer l'utilisateur à la main dans mongosh.
- **Ne pas exposer le port** : si seul le backend (même réseau Docker interne) accède à Mongo, pas de mapping `ports:` vers l'hôte. Voir la fiche reseau-interne-docker.
- **authSource oublié** : erreur classique `Authentication failed` alors que le user/password sont corrects, parce que le client cherche l'utilisateur dans la base applicative au lieu de `admin`.
- **mongosh sans -u/-p avec auth active** : la connexion passe, mais chaque commande renvoie `unauthorized`. Toujours passer `-u root -p --authenticationDatabase admin`.
