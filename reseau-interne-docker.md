---
title: Réseau interne Docker — n'exposer publiquement que ce qui doit l'être
tags: [reseau, securite, decision]
---
# Réseau interne Docker : serveur-à-serveur vs client-à-serveur

## Principe

Un serveur ne doit être **publique que si un client extérieur en a besoin**. Tout le reste vit sur le réseau interne Docker. C'est le même principe que pour les bases de données : pour un site vitrine en Node.js, aucun serveur de base de données ni de mail n'a besoin d'être exposé publiquement.

## Communication "serveur à serveur" : le réseau interne Docker

Entre conteneurs d'un même `docker-compose`, pas besoin d'exposer quoi que ce soit publiquement. Le réseau interne Docker est parfait pour faire communiquer les API entre elles.

- Dans le `docker-compose.yml`, les services partagent un réseau déclaré :

```yaml
networks:
  - internal-net
```

- Le backend se connecte aux autres services via le **nom du service** (ex. `http://backend:3000`) — Docker résout le nom sur le réseau interne.
- `expose` = visible uniquement sur le réseau Docker interne ; `ports` = publié sur le VPS, donc potentiellement public. Règle : `expose` par défaut, `ports` uniquement pour la porte d'entrée publique (souvent uniquement Caddy/nginx).

## Communication "client à serveur" : obligatoirement public, donc protégé

Le navigateur web est à l'extérieur. Il a **obligatoirement** besoin d'une URL publique (`api-demo...`) pour communiquer avec le backend. Ce qui est public doit donc être protégé :

- fail2ban (couper le bruit bruteforce) → [fail2ban.md](fail2ban.md)
- rate limiting dans Express (`express-rate-limit`) → [bruteforce-protection.md](bruteforce-protection.md)
- HTTPS via reverse-proxy → [caddy.md](caddy.md)

## Fiches liées

- [caddy.md](caddy.md) — `expose` vs `ports` en pratique, Caddy comme seule porte d'entrée
- [fail2ban.md](fail2ban.md) — bannir les IP qui martèlent
- [bruteforce-protection.md](bruteforce-protection.md) — Cloudflare + rate limiting
- [fermer-ports-firewall.md](fermer-ports-firewall.md) — le firewall Infomaniak et le port 22

## État

**Vivant**
