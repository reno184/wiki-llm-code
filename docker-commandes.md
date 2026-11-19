---
title: Commandes Docker utiles
tags: [docker, howto]
fiches-liees: []
---

# Lancer mongosh dans un conteneur

`docker exec -it mongodb_demo mongosh demo`

# Lancer un seul service d'un docker compose

`docker compose up -d` sans argument démarre **toute** la stack. Mais on peut cibler un seul service en le nommant à la fin de la commande :

```bash
# démarre uniquement le service minio (en arrière-plan), pas les autres
docker compose up -d minio
```

Utile pour tester un service isolément, relancer un conteneur après un changement, ou éviter de démarrer toute la machine quand on n'en a pas besoin.

Points à savoir :

- `up -d <service>` démarre aussi les **dépendances** déclarées dans `depends_on` (ex : lancer l'app démarre la base). Ajouter `--no-deps` pour vraiment n'avoir que ce service :

```bash
docker compose up -d --no-deps minio
```

- Si le service tourne déjà, la commande est un no-op (ou le recrée si la config/image a changé). Pour arrêter un seul service : `docker compose stop minio` (sans toucher au reste).
- Les autres commandes acceptent aussi le ciblage par service : `docker compose logs -f minio`, `docker compose pull minio`, `docker compose restart minio`.

## Exemple concret : ce que crée un lancement isolé de MinIO

Partie du docker-compose.yml (projet `proto-app`) :

```yaml
minio:
  image: cgr.dev/chainguard/minio:latest
  volumes:
    - minio_data:/data
```

Après `docker compose up -d minio`, trois objets existent, et leur nommage raconte qui a créé quoi :

| Objet | Nom créé | D'où vient le nom |
|---|---|---|
| Conteneur | `minio-1` | nom du service (`minio`) + suffixe d'instance `-1` |
| Image | `cgr.dev/chainguard/minio` | telle que déclarée dans `image:` (`:latest` est juste le tag) |
| Volume | `proto-app_minio_data` | préfixe du projet compose (`proto-app`) + nom du volume (`minio_data`) |

Lecture pratique :

- **Lancer un service isolément aide à comprendre** : on voit exactement ce que compose crée pour ce seul service, sans le bruit des autres conteneurs. `docker compose ps minio` ou `docker inspect minio-1` donnent les détails (ports, réseau, montages).
- Le préfixe `proto-app` vient du **nom du dossier** du docker-compose.yml, sauf si `name:` est défini en tête de compose ou `COMPOSE_PROJECT_NAME` dans l'env. Ça permet à plusieurs stacks de cohabiter sans collision de noms.
- Le suffixe `-1` apparaît quand le service n'a pas de `container_name:` explicite. Avec `container_name: minio` le conteneur s'appellerait `minio` tout court, mais ce nom devient unique et le service ne peut plus être scalé (`--scale minio=2` échouera).
- Retrouver le volume derrière : `docker volume ls` ou `docker inspect minio-1` (section `Mounts` : `Source` = chemin hôte, `Destination` = `/data` dans le conteneur). `docker volume rm proto-app_minio_data` pour le supprimer.
