# reCAPTCHA v3 invisible — front (Pug + JS natif)

Stack: Pug (site vitrine Node/GCP), JS natif, fetch.
Usage: génération du token côté client, sans interaction utilisateur (v3 invisible).

## Choix d'implémentation (standard du wiki)

- Le token est envoyé **dans le header** `g-recaptcha-response` de la requête fetch, pas dans le body JSON.
- La clé de site (publique) est passée dans l'URL du script de chargement.

## Code

### 1. Chargement du script — `index.html` (Pug)

```pug
script(src="https://www.google.com/recaptcha/api.js?render=CLE_PUBLIQUE" async defer)
```

### 2. Formulaire de contact — JS

```js
const token = await grecaptcha.execute('CLE_PUBLIQUE', { action: 'submit' });

const rep = await fetch('/api/contact', {
  method: 'POST',
  headers: {
    'g-recaptcha-response': token,
    'Content-Type': 'application/json' // requis pour que le back parse req.body
  },
  body: JSON.stringify(body)
});

const text = await rep.text();
```

## Notes

### `g-recaptcha-response` : le "header d'autorisation" du formulaire

Le header `'g-recaptcha-response': token` joue **exactement le même rôle que le header `authorization` dans une application web sécurisée** : comme on transmettrait un JWT via `'authorization': 'Bearer <token>'` pour prouver l'identité d'un utilisateur, on transmet ici le token reCAPTCHA via son header dédié pour **prouver que la requête provient d'un humain, pas d'un bot**. Le back le lit comme n'importe quel header (`req.get('g-recaptcha-response')`) et le valide auprès de Google — l'équivalent du `siteverify` de notre côté, comme le serait la vérification d'un JWT auprès de son émetteur.

- La clé de site est **publique par conception** (aucun risque à l'exposer). Pour la propreté, la sortir dans une variable de config plutôt que de la dupliquer en dur dans le HTML et le JS.
- Toujours ajouter `'Content-Type': 'application/json'` : sinon Express ne parse pas `req.body` (bug constaté sur la première version).
- L'`action` ('submit') doit correspondre à celle vérifiée côté back si la vérification action/score est activée (voir [../backend/recaptcha-siteverify.md](../backend/recaptcha-siteverify.md)).
- Le token expire (~2 min) et est à usage unique : exécuter `grecaptcha.execute` à la soumission, pas au chargement de page.

## État

**vivant** — implémentation réelle (site vitrine legaltech).
