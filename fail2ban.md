---
title: Fail2ban
tags: [securite, infra, howto]
---
# Fail2ban — bannir automatiquement les IP qui martèlent le serveur

**fail2ban** surveille les logs du serveur et bannit automatiquement les IP qui font trop de tentatives de connexion échouées. Tutoriel de référence : https://grafikart.fr/tutoriels/flood-ddos-fail2ban-884

## Rappel : sécuriser la clé SSH d'abord

```bash
mkdir -p ~/.ssh
mv ~/Downloads/cle_vps.pem ~/.ssh/cle_vps.pem
chmod 600 ~/.ssh/cle_vps.pem
```

## Comment ça marche, concrètement

1. Il lit en continu les fichiers de log (ex. `/var/log/auth.log` où atterrissent les tentatives SSH).
2. Quand une IP échoue X fois en Y minutes (par défaut : 5 échecs en 10 min pour SSH), il l'ajoute au firewall (iptables/nftables) avec une règle de blocage.
3. Au bout d'un temps défini (par défaut 10 min, souvent réglé sur 1h ou plus), l'IP est débannie.

## Pourquoi c'est utile alors que les mots de passe sont déjà désactivés

Avec `PasswordAuthentication no`, le brute-force ne peut **pas** réussir. fail2ban n'apporte donc pas de sécurité SSH réelle dans ce cas. Son intérêt devient surtout :

- **Couper le bruit** : sans lui, les robots du net entier martèlent le port 22 24h/24 → logs énormes, charge CPU futile. Avec lui, chaque IP qui rate 5 fois se fait couper le sifflet pendant une heure.
- **Voir qui attaque** : `sudo fail2ban-client status sshd` affiche les IP bannies.
- **Protection pour les autres services** : si un jour tu exposes un service qui a un login (interface web, panel…), fail2ban sait aussi surveiller ça.

## Installation (2 minutes sur Debian)

```bash
sudo apt update && sudo apt install fail2ban -y
```

La config par défaut surveille déjà SSH, mais le bannissement est désactivé jusqu'à ce que tu crées un jail local :

```bash
sudo tee /etc/fail2ban/jail.local <<EOF
[sshd]
enabled = true
maxretry = 5
findtime = 10m
bantime = 1h
EOF
sudo systemctl enable --now fail2ban
```

**Le heredoc `<<EOF … EOF`** : le `<<EOF` dit au shell « tout ce que je tape ensuite, jusqu'au mot `EOF` seul sur une ligne, envoie-le en entrée à la commande `tee` ». `tee /etc/fail2ban/jail.local` = écrire l'entrée dans ce fichier (avec sudo). `EOF` = marqueur de fin, convention universelle. En gros : coller tout le bloc d'un coup écrit le fichier sans éditeur.

**Si tu préfères nano (plus confortable pour lire/modifier)** :

```bash
sudo nano /etc/fail2ban/jail.local
```

Puis saisir :

```
[sshd]
enabled = true
maxretry = 5
findtime = 10m
bantime = 1h
```

Et `Ctrl+O` (sauvegarder), `Entrée`, `Ctrl+X` (quitter). **Résultat identique** : le heredoc est plus rapide pour coller, nano plus confortable pour modifier. Utilise ce qui te parle.

## Pièges

- Ne pas se tester soi-même avec 5 faux mots de passe (on se bannit soi-même).
- **Attention au vrai trafic** : si on restreint un nombre de requêtes par seconde trop bas, on peut restreindre un vrai trafic.
- **Attention à ne pas bloquer des IP de Google** (bots crawl).

## Verdict pour le VPS

**Optionnel mais recommandé** — pas critique (SSH déjà blindé par les clés), mais propre, léger, et ça évite des logs inondés.
