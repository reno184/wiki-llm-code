---
title: Express trust proxy — la vraie IP derrière Caddy/nginx
tags: [backend, securite, piege]
---
# `app.set("trust proxy", 1)` — le piège de l'IP derrière un reverse-proxy

## Le one-liner

```js
app.set("trust proxy", 1);
```

À placer dans Express quand l'API est derrière un reverse-proxy (Caddy, nginx), ce qui est le cas dès qu'on suit le pattern d'exposition publique → [reseau-interne-docker.md](reseau-interne-docker.md).

## Le pourquoi : sans ça, l'IP vue par Express est fausse

Derrière un reverse-proxy, toutes les requêtes arrivent à Express avec **l'IP du proxy**, pas celle du client. Sans `trust proxy` :

- `req.ip` = l'IP du conteneur proxy, identique pour tous les visiteurs.
- **express-rate-limit compte les requêtes du proxy au lieu du vrai client** : le limiter déclenche soit pour tout le monde en même temps, soit pour personne — dans tous les cas il ne protège plus rien.
- Les logs applicatifs sont inutilisables pour bannir ou diagnostiquer.

Le `1` = « je fais confiance à **un seul** saut de proxy » (Caddy). C'est la valeur correcte quand Express ne voit qu'un seul reverse-proxy devant lui. Si le client passe par Cloudflare en plus, il faut compter les sauts ou utiliser une config plus fine.

## Fiches liées

- [reseau-interne-docker.md](reseau-interne-docker.md) — le pattern : Caddy seule porte d'entrée publique
- [bruteforce-protection.md](bruteforce-protection.md) — le rate limiting qui dépend de `req.ip`
- [caddy.md](caddy.md) — le reverse-proxy en cause

## État

**Vivant**
