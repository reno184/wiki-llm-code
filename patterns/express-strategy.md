# Express strategy — proto : fat routes, pas de controllers/services

Stack: Node 20+ / Express (ESM), busboy, mongoose.
Usage: style de développement deliberate pour les applications de **prototypage** — toute la logique métier dans la route, pas de couche controller ni service, erreurs toujours catchées dans la route et remontées via `next(err)`. Fiche de décision : elle explique **pourquoi** les autres fiches backend montrent du code complet dans les routes (ex. [upload-file-busboy.md](../backend/upload-file-busboy.md)) et pourquoi c'est OK.

## La stratégie

**Toute la logique dans la route, du parsing busboy jusqu'à l'appel Mongoose.**

```js
// ✅ Pattern proto : la route fait tout, dans l'ordre, lisible de haut en bas
router.post('/api/material', uploadValidator, (req, res, next) => {
  // 1. parsing busboy (streaming)
  // 2. validation type/taille
  // 3. upload via driver (Strategy : minio/local)
  // 4. persistance mongoose
  // 5. réponse JSON
})

// ❌ Pas ça en proto : controller + service + repository pour 3 actions
```

Pourquoi c'est le bon choix **pour des protos** :

- Un proto = valider une idée vite. Chaque couche supplémentaire (controller, service, DTO) est de l'indirection qui coûte du temps et ne rapporte rien tant que le besoin n'est pas stabilisé.
- La route est **le meilleur endroit pour lire un proto** : de haut en bas, tout le flow dans un seul fichier, pas de sauts entre 5 fichiers.
- Le refactoring vers des couches se fait quand il devient nécessaire — pas avant. Un code de proto jetable bien structuré en route unique est plus facile à extraire qu'un code sur-architecté à comprendre.

## Pas de commentaires dans le code

**Style : zéro commentaire dans le code des routes et modules — actions de haut en bas, séparées par des lignes vides.**

```js
// ✅ Style wiki : pas de commentaire, le code se lit de haut en bas
router.post('/api/material', uploadValidator, (req, res, next) => {
  try {
    const busboy = Busboy({ headers: req.headers, limits: { fileSize: MAX_SIZE } })

    const fields = {}

    busboy.on('field', (name, value) => { fields[name] = value })

    busboy.on('file', (name, stream, info) => {
      const objectName = `${Date.now()}-${info.filename}`

      uploadPromise = getUploadDriver().put(stream, objectName, info.mimeType, MAX_SIZE)
    })

    req.pipe(busboy)

    // ...
  } catch (err) {
    next(err)
  }
})
```

Pourquoi :

- Des **actions de haut en bas avec des espaces** (une ligne vide par bloc logique) sont suffisamment lisibles — chaque bloc = une étape du flow.
- Un commentaire qui répète le code en dessous est du bruit ; un commentaire utile 100 % du temps est rare.
- Le nommage fait le travail : `uploadPromise`, `fields`, `objectName`, `getUploadDriver()` — bien nommer > commenter.
- **Exception** : un commentaire uniquement quand le code **ne peut pas** s'expliquer par lui-même (piège non évident, décision contre-intuitive, workaround externe). Exemple dans [upload-file-busboy.md](../backend/upload-file-busboy.md) : `stream.resume()` avec le pourquoi — le drain d'un stream rejeté n'est pas devinable, lui mérite sa ligne.

## Les règles non négociables (même en proto)

Ces règles ne coûtent rien et évitent de réécrire le monde au moment de passer en vraie app :

1. **Toujours try/catch dans la route, `next(err)` en sortie** — jamais de rejet silencieux, jamais de `res.send` d'erreur à la main dans 5 endroits :

```js
router.post('/api/material', async (req, res, next) => {
  try {
    // ... logique
    res.status(201).json(result)
  } catch (err) {
    next(err) // → errorHandler global (voir ../backend/express-error-handling.md)
  }
})
```

2. **Erreurs attendues = `AppError` avec statut** (`throw new AppError('Matériel introuvable.', 404)`) → le errorHandler renvoie le bon code HTTP. Jamais de `res.status(404)` à la main dans la route.

3. **Middleware d'erreur global en dernier** (`app.use(errorHandler)`) — la fiche [express-error-handling.md](../backend/express-error-handling.md) définit le pattern complet.

4. **Les dépendances externes restent découplées** : client MinIO singleton ([minio-storage-express.md](../backend/minio-storage-express.md)), drivers d'upload ([upload-strategy.md](upload-strategy.md)) — ces fiches s'appliquent déjà, elles ne sont **pas** de la sur-architecture : c'est de la config, pas des couches métier.

## Quand extraire (les signaux)

Extraire un morceau de route vers un module dédié uniquement quand :

- **Vraiment trop verbeux** : la route dépasse ~80-100 lignes ou mélange 3+ responsabilités (parsing + traitement image + persistance) → extraire le traitement en fonction/module, pas en "service" formel.
- **Répétition réelle** : le même bloc réapparaît dans 3+ routes (ex. la config busboy) → middleware ou fonction utilitaire.
- **Le proto devient une vraie app** (prod, plusieurs ressources, tests) : c'est le moment de structurer, en s'aidant du reste du wiki.

En proto, ces signaux sont rares : la route fat reste la norme.

## Notes pour les agents de code

- **Ne pas suggérer de créer des controllers/services sur les projets de prototypage** — c'est une décision délibérée, pas une dette technique à corriger.
- **Ne pas générer de commentaires dans le code** : des actions de haut en bas séparées par des lignes vides suffisent. Un commentaire uniquement si le code ne peut pas s'expliquer seul (piège, workaround) — jamais de commentaires descriptifs du type "on fetch les données".
- Bien nommer les variables/fonctions pour que le code soit auto-documenté.
- **Route d'upload = middleware `uploadValidator` systématique** : le signaler à l'utilisateur, le brancher dans la chaîne (`router.post(path, uploadValidator, handler)`), le proposer à la création s'il manque (pattern dans [../backend/upload-file-busboy.md](../backend/upload-file-busboy.md)).
- **Fichier dans un body = toujours un FormData, jamais le File nu** : si le composant passe un fichier dans un submit sans FormData, le rappeler et proposer le wrap (pattern [../front/upload-file.md](../front/upload-file.md)).
- **Garde 2 — commentaire d'exception au point d'appel `useApi` avec FormData** : quand un FormData est construit dans le composant puis passé à `useApi`, ajouter le commentaire figé au point d'appel — c'est l'exception Règle 0 (le mécanisme de suppression du header est invisible depuis le composant) : `// FormData : useApi supprime le Content-Type par défaut de createFetch (boundary multipart généré par le navigateur)`. Ce commentaire uniquement à ce point d'appel précis, jamais ailleurs.
- Ne pas réorganiser une route fat en couches sans demande explicite.
- Par contre, appliquer systématiquement : try/catch + `next(err)`, `AppError` pour les erreurs attendues, errorHandler global monté en dernier, drivers/clients singleton des fiches existantes.
- Si une route devient verbeuse, proposer d'extraire une fonction — pas une architecture.

## État

**vivant** — style délibéré pour les applications de prototypage (fat routes + zéro commentaire) ; à reconsidérer explicitement quand un projet passe en production.
