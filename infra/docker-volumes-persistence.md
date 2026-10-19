# docker-volumes-persistence — quand est-ce que je perds mes données ?

## Contexte d'usage
- Fiche pense-bête : **règle unique pour savoir si un volume survit à une commande Docker**, sans avoir à réfléchir à chaque `down`.
- Complète [docker-minio-compose.md](docker-minio-compose.md) (volume `external: true` en application) et [docker-secrets-env.md](docker-secrets-env.md).

## La règle en une phrase

> **`down -v` supprime les volumes gérés par le fichier compose. Un volume `external: true` n'est PAS géré par le fichier compose → il survit.**

- Volume **normal** (déclaré dans la section `volumes:` du compose, sans `external`) : compose l'a créé avec un préfixe (`monprojet_mongo_data`) → compose **peut le supprimer** (`down -v`).
- Volume **`external: true`** : créé hors compose (`docker volume create minio_data`), le compose ne fait que le **brancher** → compose **ne le touche jamais**.

## Tableau de survie

| Commande | Volume normal (`mongo_data`) | Volume external (`minio_data`) |
|---|---|---|
| `docker compose down` | ✅ conservé | ✅ conservé |
| `docker compose down -v` | ❌ **SUPPRIMÉ** | ✅ conservé |
| `docker system prune` | ✅ conservé (utilisé) | ✅ conservé (utilisé) |
| `docker volume prune` | ✅ conservé (utilisé) | ✅ conservé (utilisé) |
| `docker volume rm <nom>` | ❌ supprimé si non utilisé | ❌ supprimé si non utilisé |
| Suppression du dossier projet / git | ✅ conservé | ✅ conservé |

- `prune` ne supprime jamais un volume utilisé par un conteneur (même arrêté) — il détache d'abord, ou refuse.
- Les volumes vivent dans `/var/lib/docker/volumes/` (Linux), **jamais dans le dossier projet** : `rm -rf` du projet, git, etc. ne les touchent pas.
- La seule perte possible d'un external : la faire **exprès** (`docker volume rm`) ou perdre la machine.

## Comment choisir (décision produit)

- **Volume normal** = données de labo : un `down -v` accidentel les efface, repartir de zéro est acceptable.
- **`external: true`** = données importantes (fichiers uploadés) : `down -v` accidentel ne les efface pas. C'est un **bouton de sécurité** — "compose n'a pas le droit d'y toucher".

## Vérifier à tout moment

```bash
docker volume ls                              # liste tout
docker volume inspect monprojet_mongo_data    # où il vit, qui le gère
docker volume inspect minio_data              # l'external : pas de label compose
```

Un volume créé par compose a un label `com.docker.compose.*` dans `inspect` ; un `external` n'a pas ces labels (il n'appartient à aucun projet).

## État

**vivant** — pense-bête validé 19/10/2026, règle unique + tableau de survie + critère de choix normal/external.
