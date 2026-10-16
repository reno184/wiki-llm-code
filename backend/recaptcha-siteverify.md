# reCAPTCHA siteverify — back Express

Stack: Node/Express, dotenv, fetch natif. Déploiement : GCP Cloud Functions (onRequest).
Usage: vérification serveur du token reCAPTCHA v3 sur les formulaires (contact, etc.).

## Choix d'implémentation (standard du wiki)

- Le token est envoyé par le front **dans le header** `g-recaptcha-response` (comme un header `authorization`), pas dans le body JSON. Les deux sont valides côté Google ; ce wiki standardise sur le header.
- Le secret est lu via `process.env.RECAPTCHA_LEGAL_COMPLIANCE` (dotenv en local, secret GCP en prod — voir [../infra/gcp-secrets.md](../infra/gcp-secrets.md)).

## Code

```js
// routes/contact.js
const express = require('express');
const router = express.Router();

router.post('/', async (req, res) => {
  // NB : le token arrive dans un HEADER (choix standard du wiki), pas dans le body
  const response = req.get('g-recaptcha-response');

  if (!response) {
    return res.sendStatus(401);
  }

  try {
    const secret = process.env.RECAPTCHA_LEGAL_COMPLIANCE;

    const params = new URLSearchParams({ secret, response });
    const verifyRes = await fetch(
      `https://www.google.com/recaptcha/api/siteverify?${params}`,
      { method: 'POST' }
    );

    const data = await verifyRes.json();

    if (data.success) {
      // traitement métier ici
      return res.sendStatus(200);
    } else {
      return res.sendStatus(400);
    }
  } catch (error) {
    // échec réseau vers Google = échouer fermé
    return res.status(500).send(error.toString());
  }
});

module.exports = router;
```

## Setup

1. Installer dotenv et le charger au démarrage : `require("dotenv").config();`
2. Créer le secret dans GCP et le monter (voir [../infra/gcp-secrets.md](../infra/gcp-secrets.md))
3. Déclarer le secret dans l'export de la Cloud Function :

```js
exports.legalCompliance = onRequest({
  region: "europe-west1",
  secrets: ["RECAPTCHA_LEGAL_COMPLIANCE"]
}, app);
```

## Améliorations connues (non bloquantes)

- v3 : Google renvoie aussi `score` (0.0–1.0) et `action`. La version actuelle ne vérifie que `success`. Pour durcir : vérifier `data.action === 'submit'` et `data.score >= 0.5` (seuil métier à ajuster avec les logs).
- Logger les scores en base pour ajuster le seuil dans le temps.

## État

**vivant** — implémentation réelle (site vitrine legaltech, GCP).
