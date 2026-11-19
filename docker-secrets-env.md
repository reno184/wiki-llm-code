# docker-secrets-env — Secrets et .env avec Docker : périmètre d'exposition

## Contexte d'usage
- Une app conteneurisée injecte ses secrets (DB, MinIO, JWT) via des variables d'environnement. Comprendre **qui peut les lire** et réduire ce périmètre de risque.

- **Pourquoi cette fiche** : mettre des mots de passe et clés API dans un `.env` exposé via Docker est risqué car ils peuvent être lus (voir ci-dessous). L'enjeu dépend surtout du contexte de déploiement.

## Verdict selon le contexte (la règle de décision)

- **VPS sécurisé / projet perso / dev local** : compose `environment` + `.env` = acceptable, pas de problème bloquant.

- **Docker déployé chez un client** : il faut faire attention — le client (ou quiconque a un accès docker sur sa machine) peut lire les secrets. Vérifier la checklist ci-dessous, envisager Docker Secrets ou un secrets manager.

- Ce risque est aussi couvert par le skill Vibe `audit-securite-code` (checklist 1) : l'audit rappelle systématiquement de gérer les `.env` quand le docker est déployé chez un client.

## Stack

Docker / Docker Compose / Node

## Piège MinIO : les credentials naissent dans le compose, le backend doit lire les mêmes clés

Les credentials MinIO ne se créent pas dans la console : c'est le service compose qui crée le user root à partir des variables d'env au premier démarrage.

```yaml
minio:
  image: cgr.dev/chainguard/minio:latest
  restart: always
  user: root
  ports:
    - "9000:9000"
    - "9001:9001"
  environment:
    MINIO_ROOT_USER: ${MINIO_ROOT_USER}
    MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
```

Le backend Node (client minio) doit pointer sur **les mêmes valeurs** :

```js
export const minioClient = new Minio.Client({
  endPoint: '127.0.0.1',
  port: 9000,
  useSSL: false,
  accessKey: process.env.MINIO_ROOT_USER,
  secretKey: process.env.MINIO_ROOT_PASSWORD,
})
```

Conséquence : **deux `.env` contenant les mêmes clés** — un à la racine du projet (lu par docker compose pour `MINIO_ROOT_USER`/`MINIO_ROOT_PASSWORD`) et un côté backend (lu par Node au démarrage). Si les valeurs divergent : `SignatureDoesNotMatch` / 403 à chaque appel du backend alors que la console :9001 fonctionne. Symptôme classique du piège, ne pas chercher du côté du code.

Notes :

- L'image chainguard tourne en non-root par défaut, d'où le `user: root` : nécessaire pour que MinIO puisse écrire dans son volume.
- `endPoint: '127.0.0.1'` ne marche que si le backend tourne **sur l'hôte** (dev). Backend conteneurisé : utiliser le nom du service (`minio`) — cf. docker-commandes.md.
- Piège annexé : une fois le volume initialisé, changer `MINIO_ROOT_USER/PASSWORD` dans le .env **ne met pas à jour** les creds existants dans le volume. Il faut recréer le volume ou changer le mot de passe via la console/mc.

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

## todo

- Comprendre comment protéger le secret quand image docker chez un client
- Comprendre pourquoi il est possible de lire une image en clair avec docker inspect, comment crypter l'image

## État

vivant 
