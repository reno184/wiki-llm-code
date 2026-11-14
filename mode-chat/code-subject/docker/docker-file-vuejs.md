# Explication ligne par ligne

Ce Dockerfile utilise des **builds multi-stage** : 3 images successives (développement, build, production) pour une image finale légère.

```dockerfile
FROM node:20-alpine AS base
```
Image de base : Node.js 20 sur Alpine Linux (très légère, ~50 Mo). Nommée `base` pour être réutilisée.

```dockerfile
WORKDIR /app
```
Définit `/app` comme répertoire de travail. Toutes les commandes suivantes s'exécutent dedans.

---

## 🛠️ Stage 1 : développement

```dockerfile
FROM base AS development
```
Part du stage `base`. Ce stage sert au dev local (hot-reload, etc.).

```dockerfile
COPY package*.json ./
```
Copie **uniquement** `package.json` et `package-lock.json` dans `/app`. Astuce classique de cache Docker : comme les dépendances changent rarement, cette couche reste validée tant que ces fichiers ne changent pas.

```dockerfile
RUN npm install
```
Installe toutes les dépendances (y compris `devDependencies`, nécessaires pour le dev).

```dockerfile
COPY . .
```
Copie tout le reste du code source.

```dockerfile
EXPOSE 5173
```
Documente le port 5173 (port par défaut de Vite, le serveur de dev).

```dockerfile
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```
Commande au démarrage : lance le serveur de dev. `--host 0.0.0.0` est **indispensable** dans Docker — sans ça, Vite n'écoute que sur `localhost` et le port ne serait pas accessible depuis l'extérieur du conteneur.

---

## 🏗️ Stage 2 : build

```dockerfile
FROM base AS build-stage
```
Repart de `base` (image Node propre, sans le code du stage dev).

```dockerfile
COPY package*.json ./
RUN npm install
COPY . .
```
Même schéma : dépendances d'abord (cache), puis code source.

```dockerfile
RUN npm run build
```
Compile l'application de production. Pour Vite, ça produit le dossier `dist/` (HTML/JS/CSS minifiés, statiques).

---

## 🚀 Stage 3 : production

```dockerfile
FROM nginx:alpine AS production-stage
```
**Changement d'image** : on abandonne Node pour Nginx. C'est l'intérêt du multi-stage — l'image finale ne contient ni Node, ni node_modules, ni code source.

```dockerfile
COPY --from=build-stage /app/dist /usr/share/nginx/html
```
Copie **uniquement** le dossier `dist/` du stage précédent vers la racine servie par Nginx. C'est le seul artefact qui survit au build.

/usr/share/nginx/html n'existe que dans le conteneur. 
Votre Debian hôte ne voit jamais ce dossier. 
Chaque conteneur a son propre système de fichiers isolé, construit en couches à partir de l'image nginx:alpine.

```dockerfile
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
Sert sur le port 80. `daemon off;` fait tourner Nginx au premier plan — sans ça, Nginx passerait en arrière-plan et le conteneur s'arrêterait immédiatement.

---

## ⚠️ Points d'amélioration pratiques

1. **`npm ci` au lieu de `npm install`** en build-stage : installation reproductible depuis le lockfile, et plus rapide en CI.
2. **`.dockerignore`** : exclure `node_modules` et `dist` locaux, sinon `COPY . .` peut écraser l'install avec des fichiers de la machine hôte.
3. **Un seul stage build suffit** pour copier le code : les stages 1 et 2 dupliquent les mêmes 3 lignes, c'est normal ici car ils sont indépendants.
4. **Nginx et le mode history** (Vue Router/React Router) : ajoutez une config Nginx avec `try_files $uri /index.html;` sinon les routes directes renverront 404.

Note : les `\*` et `\[` dans votre texte sont des artefacts d'échappement — dans le fichier réel, c'est bien `package*.json` et `["npm", ...]`.


port 81 port du vPS
```
  frontend:
    build:
      context: ./frontend
      target: development
    ports:
      - "81:80"
```
