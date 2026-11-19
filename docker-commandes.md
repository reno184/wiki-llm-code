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
