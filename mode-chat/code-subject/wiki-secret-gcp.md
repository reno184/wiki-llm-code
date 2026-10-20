# wiki-secret-gcp — créer un secret dans GCP Secret Manager

Usage: action RARE — checklist pas-à-pas pour créer un secret dans GCP et le monter dans une Cloud Function. Mode chat Renaud, jamais chargé par un agent.

## Où trouver le menu Secret Manager

Dans la console GCP :
- Menu de gauche : **Sécurité → Secret Manager → Secret**, ou
- barre de recherche en haut de la console : taper **« secret »** pour trouver le menu.

## Création du secret

1. Créer un **secret** dont le nom de la **clé est identique à celle du `.env`** et du `.env.example` du projet (ex. `RECAPTCHA_LEGAL_COMPLIANCE`).
2. La valeur = la valeur présente dans le `.env` local.
3. Rappel: une clé = un secret, nom strictement identique entre `.env`, `.env.example` et GCP — c'est ce qui permet de monter le secret sans changer le code (`process.env.MA_CLEF` marche partout).

## Montage dans la Cloud Function

Déclarer le secret dans l'export de la fonction pour qu'il soit monté en variable d'environnement :

```js
exports.maFonction = onRequest({
  region: "europe-west1",
  secrets: ["MA_CLEF"]
}, app);
```

## Checklist ne rien oublier

- [ ] Clé présente dans `.env` (local) ET `.env.example` (squelette commité, sans valeur)
- [ ] Secret GCP créé avec **exactement le même nom de clé**
- [ ] Secret déclaré dans l'export de la Cloud Function
- [ ] `process.env.MA_CLEF` utilisé dans le code (jamais de valeur en dur)

## État

**vivant** — procédure réelle (déploiement site vitrine legaltech sur GCP).
