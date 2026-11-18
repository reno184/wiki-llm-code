## Deocekr

> Pour une simple page html

```bash
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

```bash
services:
  web:
    build: .
    expose:
      - "80"
    restart: unless-stopped
```
L'instruction EXPOSE 80 dans un Dockerfile est une documentation qui indique à Docker sur quel port le conteneur écoute à l'intérieur.
EXPOSE 80 ne rend pas le port accessible depuis l'extérieur (depuis internet ou votre VPS) par lui-même. Il dit simplement :
"Attention, l'application qui tourne dans ce conteneur (ici Nginx) écoute sur son port interne 80."


Voici ce que fait restart: unless-stopped :
Redémarrage automatique après un reboot du VPS : Si votre serveur VPS redémarre (suite à une mise à jour du système, une panne électrique ou une maintenance), Docker relancera automatiquement votre site au démarrage sans aucune intervention manuelle de votre part.
Respect de vos arrêts manuels : La condition unless-stopped ("sauf si arrêté") signifie que si vous arrêtez volontairement le conteneur vous-même avec la commande docker compose stop, Docker ne le relancera pas tout seul au redémarrage du serveur.


La commande **`CMD ["nginx", "-g", "daemon off;"]`** indique à Docker quelle commande exécuter au démarrage du conteneur pour lancer le serveur Nginx.

Voici la décomposition ligne par ligne :

### 1. `CMD`
C'est la commande par défaut que le conteneur exécute lorsqu'il démarre.

### 2. `"nginx"`
C'est tout simplement le nom du programme à exécuter : le serveur web Nginx.

### 3. `"-g", "daemon off;"` *(Le point le plus important !)*
- **Par défaut**, Nginx fonctionne sous forme de **démon** (processus qui bascule en arrière-plan).
- **Mais dans Docker**, un conteneur vit **uniquement** tant que son processus principal (le programme lancé au démarrage) reste actif au premier plan. Si le programme passe en arrière-plan, Docker pense qu'il a fini son travail et **éteint immédiatement le conteneur**.
- L'option `daemon off;` ordonne à Nginx de **rester au premier plan** (foreground).

### En résumé
Grâce à `"daemon off;"`, Nginx continue de tourner indéfiniment au premier plan, ce qui permet à votre conteneur Docker de **rester allumé et prêt à répondre aux requêtes web**.
