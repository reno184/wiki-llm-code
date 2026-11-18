---
title: Mongoose - schéma utilisateur, bcrypt, connexion DB
tags: [backend, howto]
fiches-liees: [mongodb]
---
# Mongoose

## models/User.js

```js
import bcrypt from 'bcrypt'
import mongoose, {Schema} from 'mongoose'

const SALT_WORK_FACTOR = 10 // Facteur de coût pour la complexité de Bcrypt

const _userSchema = new Schema(
    {
        email: {
            type: String,
            required: [true, "L'email est obligatoire"],
            unique: true, // Crée un index unique dans MongoDB
            lowercase: true, // Uniformise en minuscules pour éviter "Test@..." vs "test@..."
            trim: true,
        },
        password: {
            type: String,
            required: [true, 'Le mot de passe est obligatoire'],
            minlength: [8, 'Minimum 8 caractères'],
            select: false, // Sécurité : masque le mot de passe dans les find() par défaut
        },
        avatar: {
            imageUrl: {type: String, default: ''},
            blurHash: {type: String, default: ''},
        },
    },
    {timestamps: true}
)

// Hook exécuté avant la sauvegarde : ne re-hache le mot de passe que s'il a été modifié (ou créé)
_userSchema.pre('save', async function () {
    if (!this.isModified('password')) return
    const salt = await bcrypt.genSalt(SALT_WORK_FACTOR)
    this.password = await bcrypt.hash(this.password, salt)
})

// Méthode personnalisée pour comparer les mots de passe
_userSchema.methods.isValidPassword = async function (candidatePassword) {
    return await bcrypt.compare(candidatePassword, this.password)
}

export const UserModel = mongoose.model('User', _userSchema)
```

## Piège : login et `select: false`

`select: false` masque `password` dans tous les find() par défaut. Au login, si on récupère l'utilisateur sans le réinclure, `this.password` vaut `undefined` dans `isValidPassword` et la comparaison échoue silencieusement. Il faut le réinclure explicitement :

```js
const user = await UserModel.findOne({email}).select('+password')
const ok = await user.isValidPassword(candidate)
```

## Connexion MongoDB via Mongoose (config/db.js)

Backend Express (ESM), MongoDB containerisé Docker Compose. Importé dans `index.js` et appelé dans `startServer()` **avant** `app.listen()`.

```bash
npm install mongoose
```

> Pas de catch qui avale l'erreur : si `mongoose.connect` throw, l'erreur remonte au `startServer()` de `index.js` qui fait `process.exit(1)` (fail fast). Un serveur démarré sans base ne doit jamais accepter de requêtes.

```js
// config/db.js
import mongoose from 'mongoose'

export async function connectDB() {
    await mongoose.connect(process.env.MONGO_URI)
    console.log('MongoDB connecté avec succès')
}
```

```js
// index.js (pattern)
const startServer = async () => {
    try {
        await connectDB()

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

```yaml
# Service MongoDB dans docker-compose.yml
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

### Points attention

- `MONGO_URI` côté backend : hôte = **nom du service compose** (`mongodb`), pas `localhost` : `mongodb://${MONGO_USER}:${MONGO_PASSWORD}@mongodb:27017/${MONGO_DB}?authSource=admin` (authSource=admin obligatoire avec les identifiants root).
- `depends_on` seul garantit seulement que le container est démarré, pas que Mongo accepte les connexions. Pour être strict : healthcheck `mongosh --eval "db.adminCommand('ping')"` + `condition: service_healthy` chez le backend. En pratique `mongoose.connect` retry nativement au premier appel — mais le fail fast de `startServer` reste le filet.
- Volume `mongo_data` en `external: true` : résiste à `docker compose down -v` / `docker system prune`. Le créer une fois avec `docker volume create mongo_data`.
- Variables `.env` (non commité) : `MONGO_USER`, `MONGO_PASSWORD`, `MONGO_DB`.
