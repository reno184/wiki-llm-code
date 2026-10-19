# docker-minio-compose — MinIO en service Docker Compose

## Contexte d'usage
- MinIO auto-hébergé comme stockage S3. **Consommateur par défaut = API Node/Express** (voir `backend/minio-storage-express.md`). Les services Python ne concernent que le profil « Python + Docker » désigné par le project rules — le mot-clé « docker » ne déclenche jamais les fiches Python.
- Console web d'admin sur :9001.

## Service MinIO
```yaml
minio:
  image: minio/minio:latest
  restart: always
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
- Volume `minio_data` **en `external: true`** : le volume est géré hors compose → `docker compose down -v` ou `docker system prune` ne **suppriment pas** les données MinIO. Le créer une fois avec `docker volume create minio_data`.
- `depends_on` seul garantit seulement que le container est démarré, pas que MinIO est prêt. Ajouter un healthcheck et l'exprimer chez le consommateur :

```yaml
minio:
  # ... (config ci-dessus)
  healthcheck:
    test: ["CMD", "mc", "ready", "local"]
    interval: 5s
    timeout: 5s
    retries: 5

backend:
  depends_on:
    minio:
      condition: service_healthy
    mongodb:
      condition: service_started
```

## Variables à mettre dans `.env` (non commité) - valeur stocké dans doc sécurisée macOS
`MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD`

## État
vivant
