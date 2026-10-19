# docker-minio-compose — MinIO en service Docker Compose

## Contexte d'usage
- MinIO auto-hébergé comme stockage S3. **Consommateur par défaut = API Node/Express** (voir `backend/minio-storage-express.md`). Les services Python ne concernent que le profil « Python + Docker » désigné par le project rules — le mot-clé « docker » ne déclenche jamais les fiches Python.
- Console web d'admin sur :9001.

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
- **`user: root`** : l'image tourne en non-root (uid 65532) et les volumes compose appartiennent à root — sans cette ligne, permission denied sur /data.
- **Pas de healthcheck** : l'image est minimale (pas de `mc`, pas de `curl`, pas de shell). L'init bucket se fait côté code (SDK Node au démarrage du backend, idempotent — voir `minio-storage-express.md`).
- **`depends_on` côté consommateur = `service_started`** (pas de `service_healthy` possible sans healthcheck). Le backend peut démarrer avant que MinIO écoute : l'init bucket du SDK doit tolérer ça (retry au démarrage ou dépendance douce dans le code).

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

## État

**vivant** — compose réel validé 19/10/2026 (image Chainguard, `user: root`, `depends_on service_started`).
