# wiki-recaptcha — anti-bot complet (décision + front + back)

Usage: action RARE (~1 fois/an) — copier/coller le code dans le projet et adapter à la main. Mode chat Renaud, pas routé vers les agents.
Fusion des anciennes fiches : backend/recaptcha-siteverify.md + front/recaptcha-invisible-v3.md + patterns/recaptcha-choix-offre.md (supprimées).

## 1. Décision d'offre (à relire avant toute intégration)

**reCAPTCHA v3 classique** pour tous les projets actuels (sites vitrines, ~10 visites/jour).

Pourquoi pas Enterprise / Google Fraud Defense :
- **Gratuité** : v3 classique gratuit ; Enterprise/Fraud Defense implique la facturation Cloud.
- **Simplicité** : pas de Billing account obligatoire, pas de clés d'API Enterprise, pas d'IAM complexe.
- **Invisibilité** : aucun puzzle, analyse en arrière-plan via un score 0.0–1.0.

Réévaluer si trafic significatif / abuse avéré / analyses avancées. Alternative légère sans cookie Google : Cloudflare Turnstile (non retenue à ce jour).

## 2. Checklist complète — ne rien oublier

- [ ] Clé de site (publique) : dans l'URL du script de chargement
- [ ] Secret (privé) : `process.env.RECAPTCHA_LEGAL_COMPLIANCE` — dotenv en local, secret GCP monté en prod (voir [../../infra/gcp-secrets.md](../../infra/gcp-secrets.md))
- [ ] Front : script reCAPTCHA chargé dans la page
- [ ] Front : `grecaptcha.execute` **à la soumission** (token expire ~2 min, usage unique), pas au chargement
- [ ] Front : token envoyé dans le **header** `g-recaptcha-response` (standard du wiki, pas le body JSON)
- [ ] Front : `'Content-Type': 'application/json'` obligatoire, sinon Express ne parse pas `req.body` (bug constaté v1)
- [ ] Back : lecture du header `req.get('g-recaptcha-response')` → 401 si absent
- [ ] Back : siteverify auprès de Google, échouer fermé en cas d'erreur réseau (500)
- [ ] GCP : secret déclaré dans l'export de la Cloud Function
- [ ] `action` identique front/back si vérification action/score activée

Le header `'g-recaptcha-response': token` joue le même rôle que le header `authorization` avec un JWT : prouver que la requête vient d'un humain, pas d'un bot ; le back le valide auprès de Google (l'équivalent du siteverify / de la vérification d'un JWT auprès de son émetteur).

## 3. Code front (Pug + JS natif)

Chargement du script — `index.html` (Pug) :

```pug
script(src="https://www.google.com/recaptcha/api.js?render=CLE_PUBLIQUE" async defer)
```

Formulaire de contact — JS :

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

La clé de site est **publique par conception** : la sortir dans une variable de config plutôt que de la dupliquer en dur HTML/JS.

## 4. Code back (Node/Express, déploiement GCP Cloud Functions onRequest)

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

Setup back :
1. Installer dotenv et le charger au démarrage : `require("dotenv").config();`
2. Créer le secret dans GCP et le monter (voir [../../infra/gcp-secrets.md](../../infra/gcp-secrets.md))
3. Déclarer le secret dans l'export de la Cloud Function :

```js
exports.legalCompliance = onRequest({
  region: "europe-west1",
  secrets: ["RECAPTCHA_LEGAL_COMPLIANCE"]
}, app);
```

## 5. Durcissements connus (non bloquants)

- Google renvoie aussi `score` (0.0–1.0) et `action`. La version actuelle ne vérifie que `success`. Pour durcir : vérifier `data.action === 'submit'` et `data.score >= 0.5` (seuil métier à ajuster avec les logs).
- Logger les scores en base pour ajuster le seuil dans le temps.

## État

**vivant** — implémentation réelle (site vitrine legaltech, GCP).
