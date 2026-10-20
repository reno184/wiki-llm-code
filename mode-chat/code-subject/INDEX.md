# INDEX — mode-chat/code-subject

Sujets de code rares (~1×/an ou moins) : snippet complet + checklist « ne rien oublier » (secret, front, back), pour copié/collé manuel. Jamais chargé par un agent.

| Fiche | Usage | État |
|---|---|---|
| [wiki-recaptcha.md](wiki-recaptcha.md) | Anti-bot complet : décision d'offre (v3 classique vs Enterprise) + checklist (secret GCP, front header `g-recaptcha-response`, back siteverify) + code front Pug/JS + code back Express/GCP | vivant |
| [wiki-secret-gcp.md](wiki-secret-gcp.md) | Créer un secret GCP Secret Manager (menu Sécurité/secret manager/secret ou recherche « secret ») + clé identique au .env/.env.example + montage Cloud Function | vivant |
| [wiki-cloud-function-public.md](wiki-cloud-function-public.md) | Rendre une Cloud Function / Cloud Run publique : case « Allow unauthenticated » à la création, ou principal allUsers + rôle Invoker après coup ; sinon 403 côté front | vivant |
| [wiki-ollama.md](wiki-ollama.md) | Ollama complet : client desktop macOS (CLI lancé = tâche de fond `ollama serve` sur localhost:11434), setup, pattern production chez le client (Express + LangChain + host.docker.internal), modèle vision frob/qwen3.5-instruct:9b (court validé, long = échec → chunking) | vivant |
| [wiki-local-model.md](wiki-local-model.md) | Modèles locaux : frob/qwen3.5-instruct:9b (tests vision, périmètre court ✅ / long ❌), paramètres production (temperature 0, enable_thinking false), inventaire | vivant |
| [wiki-ollama-langchain.md](wiki-ollama-langchain.md) | Appeler Ollama depuis Node via LangChain : `ChatOllama` (`@langchain/ollama`), baseUrl selon contexte (localhost / host.docker.internal / nom service compose), paramètres production | vivant |
| [wiki-ollama-install-windows.md](wiki-ollama-install-windows.md) | Installer Ollama sur PC client Windows : WSL2 intégré automatique, GPU NVIDIA = driver CUDA côté Windows, limite = VRAM dédiée (≠ Mac mémoire unifiée) | vivant |
| [wiki-ollama-docker.md](wiki-ollama-docker.md) | Conteneur API → Ollama natif de l'hôte : compose avec `OLLAMA_BASE_URL=http://host.docker.internal:11434`, Ollama jamais dockerisé, URL par env | vivant |
