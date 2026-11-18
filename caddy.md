---
title: Caddy — reverse-proxy, HTTPS automatique, sslip.io
tags: [infra, hebergement, howto]
---

# Caddy + sslip.io — HTTPS automatique sur le VPS

## 1. nip.io / sslip.io — le domaine de test à zéro création de compte

Infomaniak n'offre pas de sous-domaine test gratuit en propre (domaines kSuite payants). Les astuces viennent d'ailleurs : des DNS « magiques » où **n'importe quel préfixe devant ton IP devient un nom de domaine qui pointe vers ton IP**. Zéro inscription, zéro config :

```
http://demo.203.0.113.10.nip.io   →   pointe vers 203.0.113.10
http://monsite.203.0.113.10.sslip.io  →   pareil
```

Tu tapes simplement `http://nimporte-quoi.TON-IP.nip.io` dans le navigateur, et ça atteint ton VPS. Parfait pour tester DNS + HTTPS sans rien acheter.

**Bonus : Let's Encrypt délivre de vrais certificats HTTPS** pour les domaines `*.nip.io` et `*.sslip.io` — donc tu peux tester le cadenas complet gratuitement.

## 2. Caddy — HTTPS automatique, zéro config certificat

**Caddy** est un serveur web qui obtient et renouvelle le certificat HTTPS **automatiquement**. Il gère le routage et les certificats SSL pour tout le serveur. Connecte Caddy au réseau partagé `web_net` sur lequel tes applications sont aussi connectées.

### Plan de déploiement

1. **En local (WebStorm)** : ajouter `Caddyfile` et modifier `docker-compose.yml`
2. **Livraison** via SFTP WebStorm
3. **Firewall Infomaniak** : ouvrir le port 443 (le 80 est déjà fait ✅)
4. **Lancer** : `docker compose up -d --build`
5. **Vérifier** : `https://renaud.TON-IP.sslip.io` avec le cadenas 🔒

### Étape 1 — Modifier le projet dans WebStorm

**Nouveau fichier `Caddyfile`** (New → File) :

```
renaud.TON-IP.sslip.io {
    reverse_proxy web:80
}
```

Remplace `TON-IP` par l'IPv4 réelle du VPS (ex. si ton IP est `185.99.12.34` → `renaud.185.99.12.34.sslip.io`). Le nom avant l'IP est libre : `renaud`, `demo`…

**`docker-compose.yml`** :

```yaml
services:
  web:
    build: dev-ops/domaine name
    expose:
      - "80"
    restart: unless-stopped

  caddy:
    image: caddy:2
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - caddy_data:/data
    restart: unless-stopped

volumes:
  caddy_data:
```

Ce qui change et pourquoi :

- **`web`** ne publie plus le port 80 sur le VPS (`expose` = visible uniquement dans le réseau Docker interne). Ton nginx n'est plus joignable directement, seul Caddy sert de porte d'entrée.
- **`caddy`** prend les ports 80 et 443 (deux services ne peuvent pas réclamer le port 80 tous les deux).
- Le **Caddyfile** dit : « pour ce nom de domaine, redirige le trafic vers le conteneur `web` ». En voyant un nom de domaine public, Caddy obtient automatiquement un certificat Let's Encrypt et force le HTTPS.
- **`caddy_data`** : volume qui stocke le certificat — il survit aux redémarrages, pas besoin de re-valider Let's Encrypt à chaque `up`.

À la fin : 4 fichiers — `index.html`, `Dockerfile`, `docker-compose.yml` (modifié), `Caddyfile` (nouveau).

### Recharger Caddy sans redémarrer

```bash
docker exec -it caddy caddy reload
```

## Docs

- Tutoriel nginx + SSL + Let's Encrypt : https://grafikart.fr/tutoriels/nginx-ssl-letsencrypt-747
