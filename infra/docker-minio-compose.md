# docker-minio-compose — MinIO en service Docker Compose

## Contexte d'usage
- MinIO auto-hébergé comme stockage S3 partagé entre plusieurs services (Node, Python) d'une même stack compose.
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

- Les services consommateurs déclarent `depends_on: [minio]`.

## Variables à mettre dans `.env` (non commité)
`MONGO_USER`, `MONGO_PASSWORD`, `MONGO_DB`, `JWT_SECRET`, `JWT_REFRESH_SECRET`, `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD`


## État
vivant
