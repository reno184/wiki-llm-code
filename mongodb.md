### Mongosh (terminal)

1. Se connecter `mongosh`
2. Se déconnecter `exit`
3. voir les bases de données `show dbs`
4. utiliser une base de donnée `use mabase`
5. `show collections`
6. `db.users.insertOne({ nom: "Renaud", age: 40 })`
7. `db.getCollection("testers").drop();`

await BetaTester.findOne({code}).select('+password')
``
  db.xxx.find()
  db.xxx.insertOne(),
  db.xxx.deleteOne(),
  db.xxx.insertMany(),
  db.xxx.drop()
``

``
  db.xxx.find({}, {name : 0, age :0, 'company.location':1, _id:0} )
``

``
  db.xx.find( {"categoryId": { $in: [2,3] } } )
``
``
{
    find: (params, field) => model.find(params, field).sort({ name: 1 }),
    findById: _id => model.findById(_id),
    post: body => model.create(body),
    incCounter: _id => model.updateOne({ _id }, { §inc: { count: 1 } }),
    decCounter: _id => model.updateOne({ _id }, { §inc: { count: -1 } }),
    pushArray: (_id, imageId) => model.updateOne({ _id }, { §push: { imageIds: imageId } }),
    pullArray: (_id, imageId) => model.updateOne({ _id }, { §pull: { imageIds: imageId } })
  }
``
``
$addToSet push uniquement si valeur pas presente
``


``
db.users.deleteOne({_id:ObjectId("657d6faaece4704839a3b70f")})
``

``
db.betatesters.updateOne({_id:ObjectId("6af9cc1fb27a41648de21792")},{$set : {code : "z88ZEdlaT9989kU1"}})
``

> Attention ! si model BetaTester avec Mongoose, dans la base de données la collection est minuscule et pluralize `betatesters`

### index

MongoDB **exécute toujours la requête**, même sans index adapté — simplement avec un `COLLSCAN` (scan complet de la collection), potentiellement lent sur de gros volumes

### populate

Voici comment le déclarer ObjectId sur la clef externe:

```js
studyMaterialId: {
  type: mongoose.Schema.Types.ObjectId,
  ref: 'StudyMaterial'
}
```

```javascript
const result = await WordMeaning.find().populate('studyMaterialId');
```

### data modeling

$lookup to join many-many relationship
relation one-to-one <- important
relation one-to-many <- important
operation facto, becarefull Indexe
Bucket pattern (easy)
Computed pattern
Subset pattern
Extended reference pattern (easy)

### operator
 updateMany
increment
 update one
 decrement
push and pull into array
operator $each (permet d'ajouter ou supprimer plusieurs valeurs à un tableau)
 notExist(null)


