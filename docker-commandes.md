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
