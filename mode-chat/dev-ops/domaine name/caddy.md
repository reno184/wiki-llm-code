

# sslip.io

Oui, il y a même plusieurs astuces — dont une à **zéro création de compte**. Mais pas vraiment côté Infomaniak : Infomaniak n'offre pas de sous-domaine test gratuit en propre (ils offrent des domaines via kSuite mais payants). Les astuces viennent d'ailleurs :

## 1. nip.io / sslip.io — le zéro effort (recommandé pour tester)

Ce sont des DNS « magiques » : **n'importe quel préfixe devant ton IP devient un nom de domaine qui pointe vers ton IP**. Zéro inscription, zéro config :

```
http://demo.203.0.113.10.nip.io   →   pointe vers 203.0.113.10
http://monsite.203.0.113.10.sslip.io  →   pareil
```

Tu tapes simplement `http://nimporte-quoi.TON-IP.nip.io` dans le navigateur, et ça atteint ton VPS. Parfait pour tester le DNS + HTTPS sans rien acheter.

Bonus : **Let's Encrypt délivre de vrais certificats HTTPS** pour les domaines `*.nip.io` et `*.sslip.io` — donc tu peux même tester le cadenas complet gratuitement.

## Verdict pratique

Pour ta dernière étape du plan : utilise **sslip.io ou nip.io**, et on fait tout le pipeline Domaine + DNS + HTTPS dessus — Let's Encrypt inclus, 0 €. Tu changeras juste le nom du domaine dans la config le jour où tu achèteras le vrai.

Tu veux qu'on attaque ? Prochaine étape concrète : ajouter le 443 à ton firewall, déployer un reverse-proxy (Caddy — le plus simple : HTTPS automatique) devant ton nginx, et tester `https://tonsite.TON-IP.sslip.io`. Feu vert ?

Caddy gère le routage et les certificats SSL pour tout le serveur

Connecte Caddy au réseau partagé web_net sur lequel vos applications (app_demo_a, app_demo_b) sont aussi connectées.

### caddyr reload

docker exec -it caddy caddy reload


Dernière étape ! 🏁 Objectif : `https://renaud.TON-IP.sslip.io` avec un vrai cadenas Let's Encrypt. Outil : **Caddy**, un serveur web qui obtient et renouvelle le certificat HTTPS **automatiquement** — zéro config certificat, c'est lui qui fait tout.

## Plan (feu vert à chaque étape)

1. **En local (WebStorm)** : ajouter `Caddyfile` et modifier `docker-compose.yml`
2. **Livraison** via SFTP WebStorm
3. **Firewall Infomaniak** : ouvrir le port 443 (le 80 est déjà fait ✅)
4. **Lancer** : `docker compose up -d --build`
5. **Vérifier** : `https://renaud.TON-IP.sslip.io` avec le cadenas 🔒

---

## Étape 1 — Modifier le projet dans WebStorm

**Nouveau fichier `Caddyfile`** (New → File) :

```
renaud.TON-IP.sslip.io {
    reverse_proxy web:80
}
```

Remplace `TON-IP` par l'IPv4 réelle du VPS (ex. si ton IP est `185.99.12.34` → `renaud.185.99.12.34.sslip.io`). Le nom avant l'IP est libre : `renaud`, `demo`, ce que tu veux.

**Remplacer tout `docker-compose.yml`** par :

```yaml
services:
  web:
    build: .
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
- **`caddy`** prend les ports 80 et 443 (d'où le changement — deux services ne peuvent pas réclamer le port 80 tous les deux).
- Le **Caddyfile** dit : « pour ce nom de domaine, redirige le trafic vers le conteneur `web` ». En voyant un nom de domaine public, Caddy obtient automatiquement un certificat Let's Encrypt et force le HTTPS.
- **`caddy_data`** : volume qui stocke le certificat — il survit aux redémarrages, pas besoin de re-valider Let's Encrypt à chaque `up`.

À la fin tu as 4 fichiers : `index.html`, `Dockerfile`, `docker-compose.yml` (modifié), `Caddyfile` (nouveau).

Dis-moi quand c'est prêt (et donne-moi juste confirmation que l'IP est bien remplacée dans le Caddyfile) et on passe à l'étape 2 : livraison SFTP.

