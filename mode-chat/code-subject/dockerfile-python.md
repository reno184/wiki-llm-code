# Dockerfile — image Python (script/pipeline, pas d'API)

Stack: Python 3.x, Docker, usage script/pipeline conteneurisé (pas de serveur web, pas d'API).
Usage: image Python minimaliste et sûre pour lancer des scripts/pipelines en conteneur ; cohérente avec la fiche infra/docker-secrets-env.md (secrets jamais dans l'image).

## Règle : requirements.txt vs apt-get

> **requirements.txt = ce que ton code importe ; apt-get = ce que les packages de requirements.txt appellent ou compilent en sous-main.**

- `pip install ocrmypdf` → au runtime il appelle `tesseract`, `ghostscript`, `pngquant` (binaires système, pas Python) → **apt-get**.

- `pip install` d'un package avec extension C → compile avec `g++` et les headers `-dev` (leptonica, tesseract) → **apt-get** (build-time).

- Symptôme « command not found » ou « gcc failed » au runtime/install → dépendance système manquante → **apt-get, pas pip**.

Chronologie du Dockerfile : **apt-get AVANT pip** (pip a besoin du compilateur et des headers pendant l'installation), **pip AVANT le code** (cache des layers).

## Pattern type

```dockerfile
FROM python:3.12-slim

RUN apt-get update && apt-get install -y --no-install-recommends \
    ghostscript tesseract-ocr tesseract-ocr-fra pngquant \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

RUN addgroup --system app && adduser --system --ingroup app app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY --chown=app:app . .

USER app

CMD ["python", "main.py"]
```

Points critiques :


- **apt-get : `--no-install-recommends` + purge des lists** (`rm -rf /var/lib/apt/lists/*`) → image légère.
- **requirements.txt copié seul, installé, PUIS copie du code** : le layer pip est mis en cache tant que requirements ne change pas (le changement de code ne re-télécharge pas les deps).
- **`--no-cache-dir`** : pas de cache pip dans l'image.
- **Utilisateur non-root** (USER app) : jamais de process root dans un conteneur Python.
- Outils de compilation (g++, headers -dev) gardés dans l'image = acceptable en labo ; en production → multi-stage build (étape build avec compilateur, étape runtime sans).
- **Pas de secrets dans le build** : jamais `ARG`/`ENV` avec un secret (visible via `docker inspect` et les layers, voir docker-secrets-env.md). Les secrets passent par .env/compose `secrets:` ou variables runtime.

## .dockerignore (minimum)

```
.git
__pycache__
*.pyc
.venv
.env
data/
*.pdf
*.md
```

**Pourquoi c'est critique (agent-first)** : `COPY . .` envoie **tout le contexte de build** au démon Docker avant même la première instruction. Sans .dockerignore :

- **Le build gonfle** : chaque `docker build` upload le contexte entier — PDF de test de 30 Mo, venv, cache Git → builds lents et image lourde.
- **Le cache pip est cassé plus souvent** : tout fichier non ignoré qui change invalide le layer `COPY . .` → re-copie systématique.
- **Des fichiers dangereux entrent dans l'image** : `.env` (secrets), `.git` (tout l'historique du projet), documents clients (`*.pdf`) — l'image est ensuite inspectable (`docker save` / layers), rien n'est vraiment « caché » dedans.
- `.venv` copié depuis l'hôte = incompatible de toute façon (paths absolus, binaires de la machine).
- `*.md` : la doc n'a rien à faire dans une image d'exécution.

Règle pour l'agent : **le .dockerignore se crée en même temps que le Dockerfile, jamais après coup** — un build sans .dockerignore est un build fautif, même s'il réussit.

## Compose type (script)

```yaml
services:
  pipeline:
    build: .
    env_file: .env
    volumes:
      - ./data:/app/data
    command: python main.py
```

- `env_file` pour la config locale, jamais de valeurs en dur.
- Volume pour les données persistantes (jamais dans l'image).
- Pas de ports exposés (pas d'interface, pas d'API).

## Anti-patterns

- Image full `python:3.12` (900 Mo) pour un script : slim suffit presque toujours.
- `pip install` en une seule étape avec COPY . . : cache cassé à chaque changement de code.
- Installer en root puis `USER` en fin de Dockerfile sans chown : fichiers illisibles pour l'utilisateur applicatif.
- venv copié depuis la machine hôte (incompatible, paths absolus) : installer dans l'image.
- **Dockerfile sans .dockerignore** : contexte de build gonflé, secrets/historique Git dans l'image, cache invalidé.
- **apt-get après pip ou mélangé au code** : casserait la chrono et le cache (compilateur requis PENDANT pip install).

## todo

- comprendre apt-ge, le cache le build, et une fois bien maitrisé faire une fiche mode-agent pour expliquer comment générer un docker file avec les bonne configurations

## État

**vivant** — pattern validé 2026-10 ; sections requirements vs apt-get et .dockerignore renforcées (agent-first, oct. 2026).
