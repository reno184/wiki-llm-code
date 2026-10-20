# wiki-cloud-function-public — rendre une Cloud Function / Cloud Run publique (allUsers)

Usage: action RARE (~1×/an, à la création d'un service) — sans ça, les appels front reçoivent 403. Mode chat Renaud, jamais chargé par un agent.

## Le piège à ne pas oublier

Par défaut, une Cloud Function (ou un service Cloud Run) est **privée** : seuls les comptes/membres IAM explicites peuvent l'appeler. Le front d'un site vitrine, lui, appelle l'URL publique → **erreur 403** tant que l'accès public n'est pas accordé.

## Checklist pas-à-pas

1. **À la création** du service (Cloud Function ou Cloud Run) : dans la section **Permissions**, cocher la case **« Allow unauthenticated invocations »** / l'invocation non authentifiée.
   - C'est la case à cocher qui accorde le rôle **Cloud Run Invoker** au principal **allUsers** (rôle `roles/run.invoker` sur le service).
2. **Après coup** (si oublié ou service existant) : onglet **Permissions** du service → **Ajouter un principal** → saisir **`allUsers`** → rôle **Cloud Run Invoker** (ou *Cloud Functions Invoker* selon le produit) → Enregistrer.
3. Vérifier : un `curl` (ou un appel fetch depuis le navigateur) sur l'URL du service ne doit plus renvoyer 403.

## Vérification / diagnostic 403

- Requête front → `403 Forbidden` / `PERMISSION_DENIED` : c'est presque toujours ce rôle `allUsers` manquant, pas le code.
- En console : Permissions du service → la liste des principaux doit contenir **allUsers** avec le rôle Invoker.

## Checklist ne rien oublier

- [ ] Case « Allow unauthenticated » cochée à la création, OU principal `allUsers` + rôle Invoker ajouté après coup
- [ ] `curl` de contrôle sur l'URL publique → réponse attendue (pas 403)
- [ ] Si l'API a un anti-bot : le token reCAPTCHA reste requis (cf. [wiki-recaptcha.md](wiki-recaptcha.md)) — 401 ≠ 403

## État

**vivant** — procédure réelle (déploiement site vitrine legaltech sur GCP).
