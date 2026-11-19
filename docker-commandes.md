---
title: Commandes Docker utiles
tags: [docker, howto]
fiches-liees: []
---
# Commandes de bases

## Rechercher 
les volume : docker volume ls
les logs: docker compose logs minio
les containers actifs : docker compose ps

## Lancer mongosh

`docker exec -it mongodb_demo mongosh demo`

# Lancer un seul service d'un docker compose

`docker compose up -d` sans argument démarre **toute** la stack. :

```bash
# démarre uniquement le service minio (en arrière-plan), pas les autres
docker compose up -d minio
```

Points à savoir :

- `up -d <service>` démarre aussi les **dépendances** déclarées dans `depends_on` . Ajouter `--no-deps` pour vraiment n'avoir que ce service :

```bash
docker compose up -d --no-deps minio
```

- Pour arrêter un seul service : `docker compose stop minio` (sans toucher au reste).

## Exemple concret : ce que crée un lancement isolé de MinIO

Partie du docker-compose.yml (projet `proto-app`) :

```yaml
minio:
  image: cgr.dev/chainguard/minio:latest
  volumes:
    - minio_data:/data
```

| Objet | Nom créé | D'où vient le nom |
|---|---|---|
| Conteneur | `minio-1` | nom du service (`minio`) + suffixe d'instance `-1` |
| Image | `cgr.dev/chainguard/minio` | telle que déclarée dans `image:` (`:latest` est juste le tag) |
| Volume | `proto-app_minio_data` | préfixe du projet compose (`proto-app`) + nom du volume (`minio_data`) |

Lecture pratique :

- **Lancer un service isolément aide à comprendre** : on voit exactement ce que compose crée pour ce seul service, sans le bruit des autres conteneurs. `docker compose ps minio` ou `docker inspect minio-1` donnent les détails (ports, réseau, montages).
- Le préfixe `proto-app` vient du **nom du dossier** du docker-compose.yml, sauf si `name:` est défini en tête de compose ou `COMPOSE_PROJECT_NAME` dans l'env. Ça permet à plusieurs stacks de cohabiter sans collision de noms.
- Retrouver le volume derrière : `docker volume ls` 

 endPoint selon l'origine de l'appel (piège : backend local vs backend conteneurisé)

Où le backend appelle MinIO change l'`endPoint` à utiliser. `127.0.0.1` dans un conteneur = le conteneur lui-même, pas l'hôte : l'appel échoue silencieusement ou en timeout.

| Où tourne ton backend ? | endPoint à utiliser |
|---|---|
| En local (`npm run dev` sur ta machine) | `127.0.0.1` (ou `localhost`) |
| Dans un conteneur Docker (compose) | `minio` (nom du service compose), jamais `127.0.0.1` |

Pourquoi : dans le réseau interne compose, le nom du service est résolu par le DNS interne Docker. Depuis l'hôte, ce nom n'existe pas, seuls les ports publiés (`ports:`) sont joignables via `127.0.0.1`.

Si un jour tu conteneurises aussi le backend, remets `endPoint: 'minio'`. Astuce pour gérer les deux cas avec le même code :

j`avascript
endPoint: process.env.MINIO_ENDPOINT || '127.0.0.1',
`

avec `MINIO_ENDPOINT=minio` dans le `.env` du conteneur backend (et rien en local, pour retomber sur `127.0.0.1`).
