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

## Points clés
- **`--console-address ":9001"` obligatoire** sinon la console ne démarre pas proprement.
- **Console web / UI MinIO accessible depuis le navigateur** : `http://localhost:9001`, connexion avec `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD` (stockés dans `.env` + note sécurisée macOS).
- `MINIO_ENDPOINT: minio` : dans le réseau compose, le **nom du service = hostname**. Les autres services consommateurs passent `minio:9000` via env.
- Volume `minio_data` **en `external: true`** : le volume est géré hors compose → `docker compose down -v` ou `docker system prune` ne **suppriment pas** les données MinIO. Le créer une fois avec `docker volume create minio_data`.
- Les services consommateurs déclarent `depends_on: [minio]`.

## Variables à mettre dans `.env` (non commité)
`MONGO_USER`, `MONGO_PASSWORD`, `MONGO_DB`, `JWT_SECRET`, `JWT_REFRESH_SECRET`, `MINIO_ROOT_USER`, `MINIO_ROOT_PASSWORD`

## Dev vs prod
- Le bucket n'est **pas** créé par compose : l'application le crée/idempotent au démarrage (`initBucket`). Pas besoin de job `mc mb`.
- Attention : le fichier servi publiquement dépend de la policy du bucket (cf. fiche minio-storage-express).

## État
vivant
