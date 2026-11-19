---
title: Sécurité — généralités
tags: [securite, checklist]
fiches-liees: [wiki-ssh]
---

# Sécurité — généralités

Fiche transverse. Sujets sécurité déjà couverts : [wiki-ssh.md](wiki-ssh.md) (clés SSH), [fail2ban.md](fail2ban.md) et [fermer-ports-firewall.md](fermer-ports-firewall.md) (serveur/VPS).

## Menace principale en solo : la compromission du poste

En solo, le risque n°1 n'est pas le datacenter : c'est le vol du laptop, qui ouvre l'accès à tout le reste (GitHub, GitLab, cloud, emails). D'où la checklist ci-dessous, à dérouler vite en cas de perte/vol.

## Checklist : perte ou vol du MacBook Air

### 1. Verrouiller et effacer à distance (en premier, si possible)

- **Localiser** ([icloud.com/find](https://www.apple.com/fr/icloud/find-my/)) : passer l'appareil en mode **Perdu** (verrouillage + message à l'écran), puis **Effacer** si le vol est confirmé (retire à distance les clés du trousseau, dont les tokens Git Credential Manager).
- Faire opposition / déposer plainte (numéro de série utile).

### 2. Révoquer l'accès aux dépôts : SSH keys et tokens

Clé publique retirée = la clé privée restée sur le Mac volé ne sert plus à rien.

**GitHub** :
- Clés SSH : [github.com/settings/keys](https://github.com/settings/keys) → Delete sur chaque clé (SSH keys + SSH signing keys).
- Personal access tokens : [github.com/settings/tokens](https://github.com/settings/tokens) → Delete/Revoke.
- Apps OAuth : [github.com/settings/applications](https://github.com/settings/applications) → Authorized OAuth Apps → Revoke.

**GitLab** :
- Clés SSH : [gitlab.com/-/profile/keys](https://gitlab.com/-/profile/keys) → Delete.
- Applications OAuth + tokens : [gitlab.com/-/profile/applications](https://gitlab.com/-/profile/applications) → Authorized applications → Revoke ; Personal access tokens → Revoke.
- Sessions actives : [gitlab.com/-/profile/sessions](https://gitlab.com/-/profile/sessions) → Sign out all devices.

### 3. Couper les autres accès depuis le Mac

- Tokens HTTPS stockés par Git Credential Manager : révoqués via les étapes tokens/OAuth ci-dessus (le trousseau local est nettoyé par l'effacement distant).
- Déconnecter à distance les sessions navigateur (compte Google : Security → Your devices ; Firefox : changer le mot de passe du compte coupe Sync).
- Changer le mot de passe de l'email principal, vérifier que le 2FA est actif sur les comptes critiques.

### 4. Après coup

- Régénérer une **nouvelle paire de clés SSH** (jamais réutiliser l'ancienne, probablement encore sur le Mac), la ré-ajouter sur GitHub/GitLab : voir [wiki-ssh.md](wiki-ssh.md).
- Vérifier l'activité des comptes : GitHub Security log ([github.com/settings/security-log](https://github.com/settings/security-log)), GitLab Sessions.

## Au quotidien (prévention, zéro usine à gaz)

- FileVault activé (par défaut sur Apple Silicon, vérifier dans Réglages système → Confidentialité et sécurité → FileVault).
- Apple ID avec 2FA + Touch ID.
- Sauvegardes Time Machine automatiques : le vol ne devient qu'une perte matérielle.
- Aucun secret dans un repo, toujours hors git (.env, .gitignore, voir [docker-secrets-env.md](docker-secrets-env.md)).
