---
title: Se protéger des attaques (DDoS, bots, bruteforce) sans Firebase
tags: [securite, howto]
---
# Comment se protéger des attaques (DDoS, Bots, Bruteforce) sans Firebase ?

## 1. Le Bouclier Externe : Cloudflare (le complément indispensable)

Puisque tu ne veux pas gérer de serveurs ou de configurations réseau complexes, le plus simple est de mettre **Cloudflare** (en version gratuite) devant ton application OVHcloud.

* **Comment ça marche ?** Tu achètes ton nom de domaine (chez OVHcloud par exemple) et tu le fais pointer chez Cloudflare. Cloudflare devient la « vitrine » de ton site.

* **Masquage d'IP :** personne ne connaîtra jamais l'adresse IP réelle de ton application. Les attaquants ne verront que les IP de Cloudflare.

* **Protection DDoS native :** si un robot tente de saturer ton API avec des millions de requêtes, Cloudflare bloque l'attaque avant même qu'elle atteigne ton serveur.

* **Le WAF (Web Application Firewall) :** Cloudflare bloque automatiquement les injections SQL, les attaques par force brute connues et les requêtes suspectes.

## 2. La Protection Interne : le Rate Limiting dans ton code Node.js

Pour éviter qu'un utilisateur malveillant (ou un script) bombarde ton API de requêtes pour deviner des mots de passe ou saturer ton serveur, installe un outil de **Rate Limiting** directement dans ton code Node.js.

Si tu utilises Express, le package standard s'appelle `express-rate-limit`.
