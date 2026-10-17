# docker-secrets-env — Secrets et .env avec Docker : périmètre d'exposition

## Contexte d'usage
- Une app conteneurisée injecte ses secrets (DB, MinIO, JWT) via des variables d'environnement. Comprendre **qui peut les lire** et réduire ce périmètre.

## Stack
Docker / Docker Compose / Node

## Comment les secrets sont exposés
- `environment` / `env_file` compose : visibles en clair via `docker inspect <container>` (section Env) et `docker exec <container> env`. Tout accès `docker` sur l'hôte = lecture des secrets.
- `COPY .env` dans une image (Dockerfile sans `.dockerignore`) : le fichier vit dans les **layers** de l'image. `docker history`, `docker save`, extraction des layers le révèlent. Un `RUN rm .env` dans un layer suivant **ne supprime rien** (le layer précédent le garde).
- `ARG`/`ENV` de build avec une valeur secrète : visible dans `docker history`.

## Nuance de périmètre
Celui qui peut lancer `docker inspect` a déjà un accès root de facto sur l'hôte. Le vrai objectif n'est pas de cacher les secrets à l'hôte, mais de **ne pas élargir** le périmètre : images publiées, registry partagée, exports d'image.

## Checklist par niveau d'effort
1. **Minimum (toujours)** :
   - `.dockerignore` contenant `.env`
   - Secrets injectés uniquement au **runtime** (compose `environment`/`env_file`), jamais au build
   - Jamais de secrets en `ARG`/`ENV` de build
2. **Mieux** : Docker Compose **Secrets** (`secrets:` → fichier monté `/run/secrets/<nom>`) : non visibles dans `docker inspect`, lisibles seulement par le process
3. **MinIO** : ne pas utiliser les creds root dans l'app. Créer un user de service dédié borné à un bucket :
   - `mc admin user add <alias> <svcUser> <svcPassword>`
   - `mc admin policy attach <alias> readwrite --user <svcUser>` (ou policy custom limitée au bucket)
   - Creds root réservés à la console :9001. Fuite des creds app = impact borné
4. **Prod** : secrets manager (GCP Secret Manager — cf. fiche gcp-secrets.md, Vault, SOPS) injectés au démarrage

## Verdict dev vs prod
- **Local/dev sur sa machine** : compose `environment` + `.env` = correct, risque quasi nul.
- **Image publiée / serveur / livrée à un client** : appliquer minimum (point 1) systématiquement, points 2-4 selon contexte.

## État
vivant
