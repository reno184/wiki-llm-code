# mongoose-connect — Connexion MongoDB via Mongoose (db.js)

## Contexte d'usage
- Backend Express (ESM) qui persiste dans MongoDB (containerisé Docker Compose).
- Fichier `config/db.js` (dossier `config/`, aux côtés de `config/minio.js`), importé dans `index.js` et appelé dans `startServer()` **avant** `app.listen()`.

## Stack
Node 20+ / Express (ESM) / mongoose / docker

## Install
```bash
npm install mongoose
```

## config/db.js
> Pas de catch qui avale l'erreur : si `mongoose.connect` throw, l'erreur remonte au `startServer()` de `index.js` qui fait `process.exit(1)` (fail fast, même contrat que `initBucket` dans minio-storage-express). Un serveur démarré sans base ne doit jamais accepter de requêtes.

```js
// config/db.js
import mongoose from 'mongoose'

export async function connectDB() {
    await mongoose.connect(process.env.MONGO_URI)
    console.log('MongoDB connecté avec succès')
}
```

## Appel dans index.js (pattern)
```js
const startServer = async () => {
    try {
        await connectDB()
        await initBucket('pdf-thumbnails')

        const PORT = process.env.PORT || 3000
        app.listen(PORT, '0.0.0.0', () => {
            console.log(`Serveur backend démarré sur le port ${PORT}`)
        })
    } catch (error) {
        console.error('❌ Échec du démarrage du serveur :', error)
        process.exit(1)
    }
}
```

## Service MongoDB dans docker-compose.yml
```yaml
services:
  mongodb:
    image: mongo:latest
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: ${MONGO_USER}
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD}
    volumes:
      - mongo_data:/data/db

volumes:
  mongo_data:
    external: true
```

## Points attention
- `MONGO_URI` côté backend : hôte = **nom du service compose** (`mongodb`), pas `localhost` : `mongodb://${MONGO_USER}:${MONGO_PASSWORD}@mongodb:27017/${MONGO_DB}?authSource=admin` (authSource=admin obligatoire avec les identifiants root).
- `depends_on` seul garantit seulement que le container est démarré, pas que Mongo accepte les connexions. Pour être strict : healthcheck `mongosh --eval "db.adminCommand('ping')"` + `condition: service_healthy` chez le backend. En pratique `mongoose.connect` retry nativement au premier appel — mais le fail fast de `startServer` reste le filet.
- Volume `mongo_data` en `external: true` : résiste à `docker compose down -v` / `docker system prune` (même règle que `minio_data`). Le créer une fois avec `docker volume create mongo_data`.
- Variables `.env` (non commité) : `MONGO_USER`, `MONGO_PASSWORD`, `MONGO_DB`.

## État
vivant
