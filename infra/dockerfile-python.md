# Dockerfile — image Python (script/pipeline, pas d'API)

Stack: Python 3.x, Docker, usage script/pipeline conteneurisé (pas de serveur web, pas d'API).
Usage: image Python minimaliste et sûre pour lancer des scripts/pipelines en conteneur ; cohérente avec la fiche infra/docker-secrets-env.md (secrets jamais dans l'image).

## Pattern type

```dockerfile
FROM python:3.12-slim

WORKDIR /app

RUN addgroup --system app && adduser --system --ingroup app app

COPY requirements.txt .
RUN pip install --no-cache-dir --prefix=/install -r requirements.txt \
    && rm -f /requirements.txt

COPY --chown=app:app . .

USER app

CMD ["python", "main.py"]
```

Points critiques :

- **Base slim** (pas alpine par défaut : wheels souvent absents sur musl, build plus lent ; slim = bon compromis standard).
- **requirements.txt copié seul, installé, PUIS copie du code** : le layer pip est mis en cache tant que requirements ne change pas (le changement de code ne re-télécharge pas les deps).
- **`--no-cache-dir`** : pas de cache pip dans l'image.
- **Utilisateur non-root** (USER app) : jamais de process root dans un conteneur Python.
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

## État

**vivant** — pattern validé 2026-10, à instancier au premier projet réel ; section .dockerignore renforcée (agent-first, oct. 2026).
