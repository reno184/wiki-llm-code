# INDEX — mode-chat/code-subject

Sujets de code rares (~1×/an ou moins) : snippet complet + checklist « ne rien oublier » (secret, front, back), pour copié/collé manuel. Jamais chargé par un agent.

| Fiche | Usage | État |
|---|---|---|
| [wiki-recaptcha.md](wiki-recaptcha.md) | Anti-bot complet : décision d'offre (v3 classique vs Enterprise) + checklist (secret GCP, front header `g-recaptcha-response`, back siteverify) + code front Pug/JS + code back Express/GCP | vivant |
| [wiki-secret-gcp.md](wiki-secret-gcp.md) | Créer un secret GCP Secret Manager (menu Sécurité/secret manager/secret ou recherche « secret ») + clé identique au .env/.env.example + montage Cloud Function | vivant |
| [wiki-cloud-function-public.md](wiki-cloud-function-public.md) | Rendre une Cloud Function / Cloud Run publique : case « Allow unauthenticated » à la création, ou principal allUsers + rôle Invoker après coup ; sinon 403 côté front | vivant |
| [wiki-ollama.md](wiki-ollama.md) | Généralités Ollama : client desktop macOS, CLI lancé = tâche de fond `ollama serve` sur localhost:11434, local uniquement (données sensibles OK) | vivant |
