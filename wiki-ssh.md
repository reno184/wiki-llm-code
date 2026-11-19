---
title: Gestion des clés SSH pour Git (GitLab, GitHub)
tags: [outils, securite, howto]
fiches-liees: [wiki-git, fail2ban, fermer-ports-firewall]
---

# Gérer ses clés SSH pour Git (GitLab, GitHub)

## Clé partagée entre GitLab et GitHub

Pas besoin d'une nouvelle clé pour chaque plateforme : GitHub et GitLab acceptent n'importe quelle clé publique existante. On réutilise la même paire de clés, il suffit d'ajouter la partie publique sur chaque plateforme.

### 1. Voir ses clés et récupérer la publique

```bash
ls ~/.ssh
cat ~/.ssh/id_ed25519.pub
```

- `id_ed25519.pub` si clé Ed25519 (recommandée), `id_rsa.pub` si RSA.
- Toujours ajouter le fichier `.pub` (une ligne, public), jamais la clé privée sans extension.

### 2. Ajouter la clé sur GitHub

- [github.com/settings/keys](https://github.com/settings/keys) → "New SSH key" → coller le contenu du `.pub`.
- Sur GitLab c'est l'équivalent : User Settings → SSH Keys (souvent déjà fait par le passé).

### 3. Tester la connexion

```bash
ssh -T git@github.com
```
## Multi-clés : une clé GitLab + une clé GitHub

Une seule clé suffit (voir ci-dessus), mais si on veut des clés séparées (ou qu'elles existent déjà), éviter que SSH prenne toujours la première trouvée. Générer une clé dédiée :

```bash
ssh-keygen -t ed25519 -C "github" -f ~/.ssh/id_ed25519_github
```

Puis router chaque plateforme vers sa clé dans `~/.ssh/config` :

```text
# GitLab
Host gitlab.com
  HostName gitlab.com
  User git
  IdentityFile ~/.ssh/id_ed25519

# GitHub
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
```

Sans ce fichier, SSH essaie les clés dans l'ordre et peut échouer avec `Permission denied (publickey)` alors que la bonne clé existe.


## Côté serveur (VPS)

Pour l'accès SSH à un serveur, voir [fail2ban.md](fail2ban.md) (jail sshd, bannir IP) et [fermer-ports-firewall.md](fermer-ports-firewall.md) (port 22, firewall Infomaniak).
