---
title: ALTCHA — remplacer reCAPTCHA (antispam sans Google, sans tracking)
tags: [securite, howto]
---
# ALTCHA — antispam de formulaire sans Google

**Décision** : reCAPTCHA abandonné, remplacé par [ALTCHA](https://altcha.org) — sans tracking, pas de dépendance à Google, validé côté serveur.

⚠️ **Attention version** : `"altcha-lib": "^2.9.0"` dans le `package.json`, mais l'import se fait via le sous-chemin **`/v1`** :

```js
const altcha = require("altcha-lib/v1");
```

## Principe

Le serveur génère un challenge (preuve de travail, proof-of-work) signé avec une clé HMAC. Le navigateur le résout, le serveur vérifie la solution. Pas de cookie, pas de tracking, zéro dépendance externe au runtime (le widget vient d'un CDN, la lib serveur est un simple package npm).

**`ALTCHA_HMAC_KEY` : variable d'environnement uniquement**, jamais dans le code → [wiki-variable-env.md](wiki-variable-env.md).

## Backend Express

```js
const altcha = require("altcha-lib/v1");

// Génération du challenge (testable sur http://localhost:3000/altcha-challenge)
app.get("/altcha-challenge", async (req, res) => {
    try {
        const challenge = await altcha.createChallenge({
            hmacKey: ALTCHA_HMAC_KEY,
            maxNumber: 50000,
        });
        res.json(challenge);
    } catch (err) {
        console.error("ERREUR ALTCHA DETAILLEE:", err);
        res.status(500).json({error: "Impossible de créer le challenge ALTCHA"});
    }
});

app.post("/contact", limiter, async (req, res) => {
    const {username, email, message, website, altchasite} = req.body;

    // Honeypot : un bot qui remplit ce champ invisible croit que ça marche
    if (website) return res.json({ok: true});

    if (!username || !email || !message || !altchasite) {
        return res.status(400).json({error: "Champs manquants"});
    }

    try {
        const isValid = await altcha.verifySolution(altchasite, ALTCHA_HMAC_KEY);
        if (!isValid) {
            return res.status(403).json({error: "Vérification anti-spam échouée"});
        }
    } catch (err) {
        console.error("ERREUR VERIFICATION ALTCHA:", err);
        return res.status(403).json({error: "Vérification anti-spam invalide"});
    }
    // ... traitement du message
});
```

Points clés :

- **Honeypot** : le champ `website` est invisible côté front ; un bot qui le remplit reçoit `{ok: true}` sans jamais déclencher d'erreur (il croit avoir réussi).
- **`limiter`** : rate limiting sur la route — nécessite `app.set("trust proxy", 1)` derrière Caddy → [express-trust-proxy.md](express-trust-proxy.md).
- Vérification serveur systématique : ce que fait le front n'est jamais une preuve.

## Front (Vue/Pug)

Charger le widget (CDN) et le brancher sur le formulaire :

```pug
script(src="https://cdn.jsdelivr.net/npm/altcha/dist/altcha.min.js" type="module" async defer)

// dans le formulaire
altcha-widget(challengeurl="/altcha-challenge", hidecredits)
```

Avant envoi, contrôler que le challenge est résolu :

```js
const altchaInput = document.querySelector('input[name="altcha"]');
if (!altchaInput || !altchaInput.value) {
    showToaster("Veuillez valider le Captcha", false);
    return;
}
```

Et envoyer `altchasite: altchaInput.value` dans le body du POST avec les autres champs.

## Fiches liées

- [bruteforce-protection.md](bruteforce-protection.md) — rate limiting express, complément indispensable
- [express-trust-proxy.md](express-trust-proxy.md) — sans ça, le `limiter` est cassé derrière Caddy
- [wiki-variable-env.md](wiki-variable-env.md) — la clé HMAC en variable d'env
- [form-validation.md](form-validation.md) — validation côté front

## État

**Vivant** — en place sur le formulaire de contact. Remplace [wiki-recaptcha.md](wiki-recaptcha.md) (abandonné : dépendance Google + friction UX).
