# INDEX — wiki-llm-code

Fiches perso de Renaud. Toutes **à plat à la racine du repo**, une fiche = une responsabilité. Recherche : partir d'ici, puis `search_code` plein texte si l'index ne suffit pas.

## Tags autorisés (vocabulaire fermé, 2 axes)

- **Domaine** (1-2 par fiche) : `securite`, `infra`, `docker`, `reseau`, `frontend`, `backend`, `build`, `mail`, `hebergement`, `outils`, `python`, `pdf`
- **Type d'information** (1 par fiche) : `howto` (recette), `piege` (truc qui a coûté du temps), `checklist`, `decision` (choix argumenté)

## VPS / Infra / Déploiement

| Fiche | Sujet / mots-clés |
|---|---|
| [infomaniak.md](infomaniak.md) | Hébergeur VPS, offre, console, firewall |
| [caddy.md](caddy.md) | Reverse-proxy, HTTPS automatique, Caddyfile, sslip.io/nip.io, Let's Encrypt, compose |
| [nginx.md](nginx.md) | Serveur web, reverse-proxy, config |
| [fail2ban.md](fail2ban.md) | Bannir IP, jail sshd, heredoc tee, logs inondés |
| [fermer-ports-firewall.md](fermer-ports-firewall.md) | Port 22, fausse bonne idée, firewall Infomaniak, IP box |
| [reseau-interne-docker.md](reseau-interne-docker.md) | Réseau interne Docker, expose vs ports, quoi exposer publiquement, protection API publique |
| [express-trust-proxy.md](express-trust-proxy.md) | trust proxy Express, vraie IP derrière Caddy/nginx, rate-limit cassé |
| [docker-commandes.md](docker-commandes.md) | Commandes Docker utiles, mongosh, lancer un seul service compose, up -d minio, --no-deps, depends_on, endPoint local vs conteneur (127.0.0.1 vs nom du service) |
| [config-docker-file.md](config-docker-file.md) | Configuration Dockerfile, .dockerignore par dossier de service (contexte de build), node_modules, .env |
| [docker-file-vuejs.md](docker-file-vuejs.md) | Dockerfile app Vue.js |
| [dockerfile-python.md](dockerfile-python.md) | Dockerfile Python |
| [docker-secrets-env.md](docker-secrets-env.md) | Secrets et .env dans Docker, périmètre d'exposition, docker inspect, piège MinIO : credentials créés par le compose (MINIO_ROOT_USER/PASSWORD), 2 .env identiques requis, SignatureDoesNotMatch |
| [docker-volumes-persistence.md](docker-volumes-persistence.md) | Volumes, persistance des données |
| [python-docker-labo.md](python-docker-labo.md) | Labo Python + Docker |
| [nodejs-express-env.md](nodejs-express-env.md) | Variables d'env Node.js/Express |

## Sécurité

| Fiche | Sujet / mots-clés |
|---|---|
| [wiki-securite.md](wiki-securite.md) | Généralités sécurité, checklist vol/perte MacBook, révoquer SSH keys GitHub/GitLab, Localiser mon Mac, tokens/OAuth, régénérer clés |
| [altcha-antispam.md](altcha-antispam.md) | ALTCHA, remplace reCAPTCHA, proof-of-work, honeypot, formulaire contact |
| [bruteforce-protection.md](bruteforce-protection.md) | DDoS, bots, bruteforce, Cloudflare, WAF, rate limiting, express-rate-limit |
| [wiki-secret-gcp.md](wiki-secret-gcp.md) | Secrets GCP |
| [cors.md](cors.md) | CORS, origines |

## PDF / OCR / PyMuPDF

| Fiche | Sujet / mots-clés |
|---|---|
| [ocr-gateway.md](ocr-gateway.md) | Router PDF scannés vers OCRmyPDF, probe image pleine-page |
| [ocr-on-png.md](ocr-on-png.md) | OCR via Ollama vision sur PNG, découpage zones denses |
| [pdf-text-selectionable.md](pdf-text-selectionable.md) | OCRmyPDF, couche texte invisible sur scanné |
| [pdf-to-markdown.md](pdf-to-markdown.md) | PyMuPDF4LLM, chunking par page, verdict : abandonné |
| [pdf-avoid-hallucination.md](pdf-avoid-hallucination.md) | Anti-hallucination LLM, suppression zones, garbage=3 |
| [pdf2htmlex-standby.md](pdf2htmlex-standby.md) | pdf2htmlEX en veille |
| [pymupdf-get_text.md](pymupdf-get_text.md) | Extraction texte : blocks/dic/words, clip+Rect, search_for, get_toc, licence Pro AGPL |
| [pymupdf-find_tables.md](pymupdf-find_tables.md) | Extraction tableaux PDF, find_tables |
| [pymupdf-images-position.md](pymupdf-images-position.md) | Position images/dessins, get_drawings, get_image_info |
| [pymupdf-detection-signature.md](pymupdf-detection-signature.md) | Détection signature, fingerprint, capteurs |
| [pymupdf_tech_anonymisation.md](pymupdf_tech_anonymisation.md) | Anonymisation : masquer/supprimer/remplacer |
| [anonymous-rectangle-clip.md](anonymous-rectangle-clip.md) | add_redact_annot, pourquoi le rectangle supprime le texte |
| [anonymisation-divers.md](anonymisation-divers.md) | Anonymisation, divers |
| [workflow-pdf2md.md](workflow-pdf2md.md) | Pipeline complet PDF → texte pour LLM |

## LLM locaux / Ollama

| Fiche | Sujet / mots-clés |
|---|---|
| [wiki-ollama.md](wiki-ollama.md) | Runtime Ollama macOS, généralités |
| [wiki-local-model.md](wiki-local-model.md) | Inventaire modèles, paramètres, limites long format |
| [wiki-ollama-langchain.md](wiki-ollama-langchain.md) | Appeler Ollama depuis Node via LangChain |
| [wiki-ollama-docker.md](wiki-ollama-docker.md) | Conteneur → Ollama natif hôte, host.docker.internal |
| [wiki-ollama-install-windows.md](wiki-ollama-install-windows.md) | Installer Ollama sur PC client Windows |
| [wiki-scaling-index.md](wiki-scaling-index.md) | Scaling, index |
| [wiki-cloud-function-public.md](wiki-cloud-function-public.md) | Cloud Function publique GCP |

## Frontend / Vue.js

| Fiche | Sujet / mots-clés |
|---|---|
| [vuejs-init.md](vuejs-init.md) | Init projet Vue.js |
| [vuejs-form.md](vuejs-form.md) | Formulaires Vue + DaisyUI, select required + placeholder |
| [form-validation.md](form-validation.md) | Validation de formulaires |
| [vuejs-router-menu.md](vuejs-router-menu.md) | Router, menu |
| [vuejs-stateless-smart-dumb.md](vuejs-stateless-smart-dumb.md) | Smart/dumb components |
| [vuejs-useapi.md](vuejs-useapi.md) | useApi, appels API |
| [wiki-vue-props-reactivity.md](wiki-vue-props-reactivity.md) | Réactivité props, interpolation dynamique |
| [wiki-vuejs-reset.md](wiki-vuejs-reset.md) | Shell projet Vue (Tailwind v4 + DaisyUI v5) |
| [vue-dom-manipulation.md](vue-dom-manipulation.md) | DOM vanilla dans Vue, nextTick, bubbling, v-if pièges |
| [modal-daisyui-vuejs.md](modal-daisyui-vuejs.md) | Modal DaisyUI + Vue |
| [vue-gsap-draw-rectangle.md](vue-gsap-draw-rectangle.md) | GSAP GreenSock Vue, overlay rectangle sélection animé, bounding box, getBoundingClientRect non réactif, triggerRef, sans deep:true |
| [daisyui-components.md](daisyui-components.md) | Composants DaisyUI |
| [wiki-tailwindcss.md](wiki-tailwindcss.md) | Tailwind safelist, classes dynamiques invisibles au build |
| [wiki-tailwindcss-grid-bootstrap.md](wiki-tailwindcss-grid-bootstrap.md) | Tailwind v4, reproduire grid Bootstrap |
| [inspiration-html-css.md](inspiration-html-css.md) | Mises en page HTML/CSS à réutiliser, footer formulaire |
| [wiki-font-awesome.md](wiki-font-awesome.md) | Font Awesome |
| [wiki-webcomponents-stack.md](wiki-webcomponents-stack.md) | Stack web components |
| [wiki-purejs.md](wiki-purejs.md) | JS natif, proxy |
| [ux-motion-adoption.md](ux-motion-adoption.md) | UX, motion, adoption |

## Backend / Node.js / Mongo

| Fiche | Sujet / mots-clés |
|---|---|
| [express-error-handling.md](express-error-handling.md) | Gestion d'erreurs Express |
| [example-error-management.md](example-error-management.md) | Exemples gestion d'erreurs |
| [wiki-mongoose.md](wiki-mongoose.md) | Mongoose : schéma utilisateur (bcrypt, hook pre save, isValidPassword), piège select:+password au login, connexion DB config/db.js fail fast, compose service mongodb, authSource=admin |
| [mongodb.md](mongodb.md) | Commandes mongosh, CRUD, opérateurs $inc/$push/$addToSet, COLLSCAN index, populate ObjectId, data modeling patterns |
| [upload-file-busboy.md](upload-file-busboy.md) | Upload fichiers, busboy |
| [wiki-upload-dragdrop-pdf.md](wiki-upload-dragdrop-pdf.md) | Upload drag & drop de PDF |
| [wiki-minio-pdf.md](wiki-minio-pdf.md) | MinIO, stockage PDF |
| [wiki-download-blob.md](wiki-download-blob.md) | Download blob |
| [wiki-variable-env.md](wiki-variable-env.md) | Variables d'env, build vs runtime Vite |
| [wiki-webstorm-http.md](wiki-webstorm-http.md) | Requêtes HTTP côté WebStorm |
| [webstorm-http-upload.md](webstorm-http-upload.md) | Upload HTTP WebStorm |

## Outils dev / Divers

| Fiche | Sujet / mots-clés |
|---|---|
| [wiki-git.md](wiki-git.md) | Git |
| [wiki-ssh.md](wiki-ssh.md) | Clés SSH Git, clé partagée GitLab/GitHub, ajout clé publique, test ssh -T, remote HTTPS → SSH, multi-clés ~/.ssh/config, passphrase trousseau macOS |
| [wiki-python.md](wiki-python.md) | Python, debug |
| [python-venv-macos.md](python-venv-macos.md) | venv, Python système vs virtuel, Mac M4, lancer API Flask port 5000 en ligne de commande, flèche verte WebStorm ne marche pas, python3.12 Homebrew, PEP 668 |
| [checklist-initialize-project.md](checklist-initialize-project.md) | Checklist initialisation projet |
| [webstorm-settings-ai.md](webstorm-settings-ai.md) | Réglages IA WebStorm |
| [webstorm-agents-junie-vibe.md](webstorm-agents-junie-vibe.md) | Agents WebStorm, Junie, Vibe |
| [mistral-vibe-cli.md](mistral-vibe-cli.md) | Vibe CLI terminal, agent custom avis rapide, lenteur 3 min boucle d'outils, ~/.vibe/agents/, active_model nom API invalide, Shift+Tab |
| [python-flask-serveur-interne.md](python-flask-serveur-interne.md) | Flask dev server en prod, micro-service interne PyMuPDF, CPU-bound, GIL vs threads, gunicorn workers, réseau docker interne |
| [webstorm-gitignore-run-configurations.md](webstorm-gitignore-run-configurations.md) | Gitignore .idea/*, réinclusion !.idea/runConfigurations/, configs debug Front-end/Back-end versionnées, piège slash final |
| [wiki-due-diligence-ma.md](wiki-due-diligence-ma.md) | Due diligence M&A |
| [stages.md](stages.md) | Stages |
