
Le Dockerfile ne contient plus que 2 étapes simples :
1. Compiler l'application Vue (npm run build).
2. Transférer le dossier dist vers nginx:alpine pour servir le site.


Pourquoi conserver Alpine ?
Oui, Alpine est fortement recommandé, même si ce n'est pas strictement obligatoire. Voici pourquoi :
Poids ultra-léger :
Une image basée sur Alpine (node:20-alpine ou nginx:alpine) est 5 à 10 fois plus petite qu'une image basée sur Debian classique.
Résultat : le transfert via SFTP / déploiement sur votre VPS est beaucoup plus rapide et consomme très peu d'espace disque.
Économie de RAM et CPU :
Alpine ne charge que le strict minimum, ce qui laisse plus de ressources libres pour votre VPS et vos futurs conteneurs backend.
Sécurité accrue :
Moins de paquets préinstallés signifie une surface d'attaque beaucoup plus réduite.

> donc pour le build du front end il me nodejs donc j'utilise node:20-alpine, par contre pour le stage production donc nginx sert les fichires frontend buildé je n'ai pas besoin de
node:20-alpine j'utilise uni nginx alpine

L'instruction EXPOSE 80 dans un Dockerfile est une documentation qui indique à Docker sur quel port le conteneur écoute à l'intérieur.

### unless-stopped
Voici ce que fait restart: unless-stopped :
1. Redémarrage automatique après un reboot du VPS : Si votre serveur VPS redémarre (suite à une mise à jour du système, une panne électrique ou une maintenance), Docker relancera automatiquement votre site au démarrage sans aucune intervention manuelle de votre part.
2. Gestion des pannes (crash) : Si le conteneur s'arrête de manière inattendue suite à un bogue, Docker essaiera immédiatement de le redémarrer.
3. Respect de vos arrêts manuels : La condition unless-stopped ("sauf si arrêté") signifie que si vous arrêtez volontairement le conteneur vous-même avec la commande docker compose stop, Docker ne le relancera pas tout seul au redémarrage du serveur.
C'est une bonne pratique standard pour tous les conteneurs destinés à la production.

### doemon off

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

### dockerignore

.dockerignore recommandé
le docker ignore ne sert à rien car je vais livrer sur le VPS uniquement les fichiers dont à besoin docker pour builder

