# docker-minio-compose — MinIO en service Docker Compose

## Contexte d'usage
- MinIO auto-hébergé comme stockage S3. **Consommateur par défaut = API Node/Express** (voir `backend/minio-storage-express.md`). Les services Python ne concernent que le profil « Python + Docker » désigné par le project rules — le mot-clé « docker » ne déclenche jamais les fiches Python.
- Console web d'admin sur :9001.

## ⚠️ Piège majeur (octobre 2026) — l'image Docker Hub n'existe plus

**`minio/minio` et `minio/mc` ont été supprimées de Docker Hub le 11/09/2026** (MinIO a retiré son édition communautaire, repo archivé en février) → `pull access denied for minio/minio`. **quay.io (`quay.io/minio/minio`) exige un login depuis le 24/09/2026**, toutes tags (compte Red Hat + robot account — usine à gaz, à éviter). L'image **Chainguard `cgr.dev/chainguard/minio` se pull anonymement, sans compte** : c'est le choix par défaut.

Le serveur, la console :9001, l'API S3 et le SDK Node fonctionnent **identiquement** avec l'image Chainguard. Si un agent voit `minio/minio` ou `quay.io/minio` dans un compose : signaler ce piège et appliquer `cgr.dev/chainguard/minio:latest`.

## Service MinIO
```yaml
minio:
  image: cgr.dev/chainguard/minio:latest
  restart: always
  user: root
  ports:
    - "9000:9000"  # API S3
    - "9001:9001"  # Console web
  environment:
    MINIO_ROOT_USER: ${MINIO_ROOT_USER}
    MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
  volumes:
    - minio_data:/data
  command: server /data --console-address ":9001"
```

## Points attention
- **Image Chainguard = minimale** : pas de `/bin/sh`, pas de `curl`, pas de `mc` dans le conteneur → **pas de healthcheck `mc ready local`** (le binaire n'y est pas), pas de `docker exec /bin/sh`. L'init bucket se fait côté code (SDK Node au démarrage du backend, idempotent — voir `minio-storage-express.md`).
- **`user: root` obligatoire** : l'image tourne en uid 65532 (non-root) et les volumes créés par compose appartiennent à root → sans cette ligne, permission denied sur /data.
- **`depends_on` côté consommateur = `service_started`** (plus de `service_healthy` possible sans healthcheck). Le backend peut démarrer avant que MinIO écoute : l'init bucket du SDK doit tolérer ça (retry au démarrage ou dépendance douce dans le code).

```yaml
backend:
  depends_on:
    minio:
      condition: service_started
    mongodb:
      condition: service_healthy
```

- Volume `minio_data` **en `external: true`** : le volume est géré hors compose → `docker compose down -v` ou `docker system prune` ne **suppriment pas** les données MinIO. Le créer une fois avec `docker volume create minio_data` — sinon `external volume "minio_data" not found` au `up`. En labo, `external: true` peut être retiré pour laisser compose gérer le volume.
- `MINIO_ROOT_USER`/`MINIO_ROOT_PASSWORD` passés aussi au backend : le SDK Node se connecte avec les mêmes credentials.

## Variables à mettre dans `.env` (non commité) - valeur stocké dans doc sécurisée macOS
`MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD`

## Migration future (documentée, pas urgent)

L'édition communautaire MinIO est **gelée pour toujours** (repo archivé, plus aucun correctif). L'image Chainguard fait le job en labo. Le jour où un besoin réel apparaît (prod, correctif sécurité), les candidats S3-compatibles : **SeaweedFS** ou **Garage** — même API S3, le code Node (SDK `minio`) ne change pas, seulement l'image et l'endpoint. Voir `patterns/upload-strategy.md` : un futur driver = une classe + la factory, zéro refonte.

## État

**vivant** — compose réel validé 19/10/2026 avec l'image Chainguard (après retrait Docker Hub + verrouillage quay) ; `user: root` + `depends_on service_started` testés.
