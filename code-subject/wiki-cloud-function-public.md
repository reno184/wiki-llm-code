# wiki-cloud-function-public — rendre une Cloud Function / Cloud Run publique (allUsers)

Usage: action RARE (~1×/an, à la création d'un service) — sans ça, les appels front reçoivent 403. Mode chat Renaud, jamais chargé par un agent.

## Le piège à ne pas oublier

Par défaut, une Cloud Function (ou un service Cloud Run) est **privée** : seuls les comptes/membres IAM explicites peuvent l'appeler. Le front d'un site vitrine, lui, appelle l'URL publique → **erreur 403** tant que l'accès public n'est pas accordé.

**À la création** du service (Cloud Function ou Cloud Run) : dans la section **Permissions**, cocher la case **« Allow unauthenticated invocations »** / l'invocation non authentifiée.
   - C'est la case à cocher qui accorde le rôle **Cloud Run Invoker** au principal **allUsers** (rôle `roles/run.invoker` sur le service).



## État

**vivant** — procédure réelle (déploiement site vitrine legaltech sur GCP).
