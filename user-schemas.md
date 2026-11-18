---
title: Schéma utilisateur Mongoose (bcrypt, validation)
tags: [backend, howto]
---
# Schéma utilisateur Mongoose

```js
// models/User.js
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
