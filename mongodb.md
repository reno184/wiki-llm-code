---
title: MongoDB - commandes mongosh, populate, data modeling
tags: [backend, howto]
fiches-liees: [user-schemas]
---
# MongoDB

## Mongosh (terminal)

1. Se connecter `mongosh`
2. Se déconnecter `exit`
3. Voir les bases de données `show dbs`
4. Utiliser une base de données `use mabase`
5. `show collections`
6. `db.users.insertOne({ nom: "Renaud", age: 40 })`
7. `db.getCollection("testers").drop();`

### CRUD

```js
db.xxx.find()
db.xxx.insertOne()
db.xxx.deleteOne()
db.xxx.insertMany()
db.xxx.drop()
```

```js
// projection : exclure des champs, inclure un champ imbriqué
db.xxx.find({}, {name: 0, age: 0, 'company.location': 1, _id: 0})
```

```js
db.xx.find({"categoryId": {$in: [2, 3]}})
```

```js
db.users.deleteOne({_id: ObjectId("657d6faaece4704839a3b70f")})
```

```js
db.betatesters.updateOne({_id: ObjectId("6af9cc1fb27a41648de21792")}, {$set: {code: "z88ZEdlaT9989kU1"}})
```

> Attention ! Avec un modèle Mongoose `BetaTester`, la collection dans la base est minuscule et pluralisée : `betatesters`.

### Opérateurs Mongoose courants

```js
{
    find: (params, field) => model.find(params, field).sort({name: 1}),
    findById: _id => model.findById(_id),
    post: body => model.create(body),
    incCounter: _id => model.updateOne({_id}, {$inc: {count: 1}}),
    decCounter: _id => model.updateOne({_id}, {$inc: {count: -1}}),
    pushArray: (_id, imageId) => model.updateOne({_id}, {$push: {imageIds: imageId}}),
    pullArray: (_id, imageId) => model.updateOne({_id}, {$pull: {imageIds: imageId}})
}
```

- `$addToSet` : push uniquement si la valeur n'est pas déjà présente
- `$each` : ajouter/supprimer plusieurs valeurs d'un tableau
- `updateMany`, `$set`, incrément/décrément, push/pull dans un tableau, `$exists` /`null`

## Index

MongoDB **exécute toujours la requête**, même sans index adapté — simplement avec un `COLLSCAN` (scan complet de la collection), potentiellement lent sur de gros volumes.

## Populate

Déclarer l'ObjectId sur la clef externe :

```js
studyMaterialId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'StudyMaterial'
}
```

```js
const result = await WordMeaning.find().populate('studyMaterialId')
```

## Data modeling

- `$lookup` pour joindre une relation many-many
- relation one-to-one : important
- relation one-to-many : important
- attention aux index (fan-out)
- Bucket pattern (easy)
- Computed pattern
- Subset pattern
- Extended reference pattern (easy)
