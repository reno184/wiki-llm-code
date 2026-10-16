# Choix d'offre : reCAPTCHA v3 classique vs Enterprise / Google Fraud Defense

Usage: décision à relire avant toute nouvelle intégration anti-bot. Décision métier, pas technique.

## Décision (version vivante)

**reCAPTCHA v3 classique** pour tous les projets actuels (sites vitrines, ~10 visites/jour).

## Pourquoi

Google pousse fortement vers **Google Fraud Defense** et **reCAPTCHA Enterprise** pour monétiser son écosystème Cloud et cibler les entreprises. Pour des sites vitrines à faible trafic, ces offres sont inutilement complexes :

- **Gratuité** : reCAPTCHA v3 classique est gratuit ; Enterprise/Fraud Defense implique la facturation Cloud.
- **Simplicité** : aucun compte de facturation Google Cloud (Billing account) obligatoire, pas de clés d'API Enterprise, pas de permissions IAM complexes.
- **Invisibilité** : aucun puzzle à résoudre, l'analyse se fait en arrière-plan via un score de 0,0 à 1,0.

## Quand réévaluer

- Trafic significatif / abuse avéré / besoin d'analyses avancées → réévaluer Enterprise.
- Alternative légère sans cookie Google : Cloudflare Turnstile (non retenue à ce jour).

## État

**vivant** — décision prise pour le site vitrine legaltech, applicable à tous les projets similaires.
