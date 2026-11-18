
```js
// models/User.js
import mongoose from 'mongoose';
import bcrypt from 'bcrypt'
const userSchema = new mongoose.Schema(
    {
        firstName: {
            type: String,
            required: [true, 'Le prénom est requis'],
            trim: true,
        },
        lastName: {
            type: String,
            required: [true, 'Le nom est requis'],
            trim: true,
        },
        email: {
            type: String,
            required: [true, "L'email est requis"],
            unique: true,
            lowercase: true,
            trim: true,
            match: [/^\S+@\S+\.\S+$/, 'Email invalide'],
        },
        password: {
            type: String,
            required: [true, 'Le mot de passe est requis'],
            minlength: [8, 'Minimum 8 caractères'],
            select: false, // exclu des requêtes par défaut
        },
        role: {
            type: String,
            enum: ['user', 'admin'],
            default: 'user',
        },
        isActive: {
            type: Boolean,
            default: true,
        },
    },
    {
        timestamps: true, // ajoute createdAt et updatedAt automatiquement
    }
);

userSchema.pre('save', async function () {
    if (!this.isModified('password')) return;
    this.password = await bcrypt.hash(this.password, 10);
});

userSchema.methods.correctPassword = async function (candidatePassword, userPassword) {
    return await bcrypt.compare(candidatePassword, userPassword);
};

export const User = mongoose.model('User', userSchema);

```
