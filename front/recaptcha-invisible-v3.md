# reCAPTCHA v3 invisible — front (Pug + JS natif)

Stack: Pug (site vitrine Node/GCP), JS natif, fetch.
Usage: génération du token côté client, sans interaction utilisateur (v3 invisible).

## Choix d'implémentation (standard du wiki)

- Le token est envoyé **dans le header** `g-recaptcha-response` de la requête fetch (équivalent d'un header `authorization`), pas dans le body JSON.
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

- La clé de site est **publique par conception** (aucun risque à l'exposer). Pour la propreté, la sortir dans une variable de config plutôt que de la dupliquer en dur dans le HTML et le JS.
- Toujours ajouter `'Content-Type': 'application/json'` : sinon Express ne parse pas `req.body` (bug constaté sur la première version).
- L'`action` ('submit') doit correspondre à celle vérifiée côté back si la vérification action/score est activée (voir [../backend/recaptcha-siteverify.md](../backend/recaptcha-siteverify.md)).
- Le token expire (~2 min) et est à usage unique : exécuter `grecaptcha.execute` à la soumission, pas au chargement de page.

## État

**vivant** — implémentation réelle (site vitrine legaltech).
