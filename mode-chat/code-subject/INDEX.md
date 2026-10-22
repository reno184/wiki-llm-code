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
| [wiki-ollama-install-windows.md](wiki-ollama-install-windows.md) | Installer Ollama sur PC client Windows : WSL2 intégré automatiquement, GPU NVIDIA = driver CUDA côté Windows, limite = VRAM dédiée (≠ Mac mémoire unifiée) | vivant |
| [wiki-ollama-docker.md](wiki-ollama-docker.md) | Conteneur API → Ollama natif de l'hôte : compose avec `OLLAMA_BASE_URL=http://host.docker.internal:11434`, Ollama jamais dockerisé, URL par env | vivant |
| [wiki-git.md](wiki-git.md) | Config Git machine : `pull.rebase true` (historique linéaire, rebase auto au pull), `rebase.autoStash true`, erreurs types (stash, conflit, abort), niveaux de config | vivant |
| [wiki-variable-env.md](wiki-variable-env.md) | Variables d'environnement : backend dotenv (`process.env`, fail fast, `.env.example`) + front Vite — mise à jour au lancement via scripts `package.json` (`VITE_*`) ou `define` dans `vite.config.js` (constante globale, lecture directe `__XXX__` ≠ `import.meta.env`) | vivant |
| [wiki-fontawesome-chat.md](wiki-fontawesome-chat.md) | Font Awesome perso : versions 6.4.0 Pro (ancien compte) / 7.3.1 Free, setup Vue (assets/css/all.min.css + webfonts, `@import` dans main.css), styles fa/fad/fal | vivant |
| [wiki-upload-dragdrop-pdf.md](wiki-upload-dragdrop-pdf.md) | Upload drag & drop PDF : useDropZone + label input caché, useApi post FormData blob, useObjectUrl pour l'aperçu | vivant |
| [wiki-minio-pdf.md](wiki-minio-pdf.md) | MinIO : config singleton + démarrage (initBucket après connectDB), upload PDF/WebP Mongoose, hook deleteOne suppression cascade, docker compose | vivant |
| [upload-file-busboy.md](upload-file-busboy.md) | Upload backend Express + busboy : middleware `uploadValidator`, pattern minimal route, contrôles optionnels (types 500, poids 400, fichier unique, champs + Joi), métadonnées regex nom de fichier | vivant |
| [wiki-vueuse.md](wiki-vueuse.md) | Composables VueUse rares : useDropZone (ref sur le div, onDrop = même onChange que le label input, filtrage type ≠ drag), useObjectUrl (Blob → URL affichable, révocation auto), useApi blob (immediate false + execute), presse-papier useClipboard COPIER (legacy true, ClipBoardBase) ≠ useEventListener paste COLLER image → FormData POST busboy | vivant |
| [wiki-webstorm-http.md](wiki-webstorm-http.md) | Client HTTP WebStorm (fichiers `.http`) : chaîner requêtes via handler de réponse `> {% client.global.set(...) %}` puis `{{var}}` (ex. `group` d'un upload step1 réutilisé dans le GET suivant), pièges (ordre d'exécution, panneau Variables, env vs global) | vivant |
