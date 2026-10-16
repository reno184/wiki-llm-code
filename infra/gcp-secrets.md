# Secrets GCP — création et montage dans une Cloud Function

Stack: GCP Cloud Functions (onRequest), Node/Express.
Usage: stocker les secrets applicatifs (clés API reCAPTCHA, SMTP, etc.) hors du code et hors du repo.

## Principe

1. En local : le secret vit dans un fichier `.env` (non commité, dans `.gitignore`), chargé par dotenv.
2. En prod (GCP) : le secret vit dans **Secret Manager**, et est monté dans la fonction via la déclaration `secrets:`.

## Étapes

### 1. Local — `.env` (non commité)

```
RECAPTCHA_LEGAL_COMPLIANCE=<secret api key du recaptcha>
```

Chargement au démarrage :

```js
require("dotenv").config();
```

### 2. GCP — créer le secret (Secret Manager)

- Console GCP → **Secret Manager** → **Create secret**
- Nom : ex. `RECAPTCHA_LEGAL_COMPLIANCE`
- Valeur : la clé secrète
- Vérifier que le compte de service de la Cloud Function a le rôle *Secret Manager Secret Accessor* sur ce secret

### 3. GCP — monter le secret dans la Cloud Function

Lecture côté code (identique local/prod) :

```js
const secret = process.env.RECAPTCHA_LEGAL_COMPLIANCE;
```

Déclaration dans l'export de la fonction :

```js
exports.legalCompliance = onRequest({
  region: "europe-west1",
  secrets: ["RECAPTCHA_LEGAL_COMPLIANCE"]
}, app);
```

## Notes

- Jamais de secret dans le repo, le front, ou une variable "publique" du code.
- Un secret monté via `secrets:` est exposé comme variable d'environnement → le même `process.env.X` fonctionne partout.
- Clé **publique** (site key reCAPTCHA) ≠ clé **secrète** : seule la seconde passe par Secret Manager.

## État

**vivant** — pattern appliqué sur le site vitrine legaltech.
