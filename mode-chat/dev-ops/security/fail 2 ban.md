### Attaque de déni-de-service

Fail2ban — bloque automatiquement les IP après quelques échecs.


https://grafikart.fr/tutoriels/flood-ddos-fail2ban-884


```
mkdir -p ~/.ssh
mv ~/Downloads/cle_vps.pem ~/.ssh/cle_vps.pem
chmod 600 ~/.ssh/cle_vps.pem
```


**fail2ban** est un petit logiciel qui surveille les logs de ton serveur et bannit automatiquement les IP qui font trop de tentatives de connexion échouées.

## Comment ça marche, concrètement

1. Il lit en continu les fichiers de log (ex. `/var/log/auth.log` où atterrissent les tentatives SSH).
2. Quand une IP échoue X fois en Y minutes (par défaut : 5 échecs en 10 min pour SSH), il l'ajoute au firewall (iptables/nftables) avec une règle de blocage.
3. Au bout d'un temps défini (par défaut 10 min, souvent réglé sur 1h ou plus), l'IP est débannie.

## Pourquoi c'est utile alors que tes mots de passe sont déjà désactivés

Tu as raison de tiquer : avec `PasswordAuthentication no`, le brute-force ne peut **pas** réussir. fail2ban n'apporte donc pas de sécurité SSH réelle dans ton cas. Son intérêt devient surtout :

- **Couper le bruit** : sans lui, les robots du net entier martèlent ton port 22 24h/24 → logs énormes, charge CPU futile, pour rien. Avec lui, chaque IP qui rate 5 fois se fait couper le sifflet pendant une heure.
- **Voir qui attaque** : `sudo fail2ban-client status sshd` affiche les IP bannies — utile pour savoir ce qui se passe sur ton serveur.
- **Protection pour les autres services** : si un jour tu exposes un service qui a un login (interface web, panel, etc.), fail2ban sait aussi surveiller ça.

## Installation (2 minutes sur Debian)

```bash
sudo apt update && sudo apt install fail2ban -y
```

La config par défaut surveille déjà SSH, mais par défaut le bannissement est désactivé jusqu'à ce que tu crées un jail local :

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

## Verdict pour ton VPS

**Optionnel mais recommandé** — pas critique (ton SSH est déjà blindé par les clés), mais c'est propre, léger, et ça évite des logs inondés. C'est la dernière ligne de la checklist sécurité de ton [Wiki VPS](knowledge://knowledge/wiki-vps/KNOWLEDGE.md). Dis-moi si tu l'installes et je coche l'étape « Sécurisation de base » dans le plan.

Bonne question — ce n'est pas une commande, c'est un **marqueur**.

## Le heredoc `<<EOF ... EOF`

Quand tu écris :

```bash
sudo tee /etc/fail2ban/jail.local <<EOF
[sshd]
enabled = true
maxretry = 5
findtime = 10m
bantime = 1h
EOF
```

Le `<<EOF` dit au shell : « tout ce que je tape ensuite, jusqu'à ce que je retape le mot `EOF` sur une ligne seul, envoie-le en entrée à la commande `tee` ».

- `tee /etc/fail2ban/jail.local` = écrire l'entrée dans ce fichier (avec sudo pour avoir les droits root).
- `EOF` = "End Of File", la balise de fin. C'est juste une convention — tu pourrais écrire `<<FIN` et finir par `FIN`, ça marcherait pareil. `EOF` est le marqueur universellement utilisé.

En gros : **coller tout le bloc d'un coup dans le terminal** écrit le fichier sans passer par un éditeur.

## Si tu préfères nano (plus simple pour débuter)

Fais exactement pareil, mais avec ton réflexe :

```bash
sudo nano /etc/fail2ban/jail.local
```

Puis tape ces 5 lignes dedans :

```
[sshd]
enabled = true
maxretry = 5
findtime = 10m
bantime = 1h
```

Et `Ctrl+O` (sauvegarder), `Entrée`, `Ctrl+X` (quitter).

**Résultat identique.** Le heredoc est juste plus rapide quand on te donne du texte à coller ; nano est plus confortable pour lire/modifier. Utilise ce qui te parle le plus.


C'est fait — la fiche [Wiki VPS](knowledge://knowledge/wiki-vps/KNOWLEDGE.md) (sous-fiche sécurité) contient maintenant :

- L'explication complète de `PasswordAuthentication no` : pourquoi c'est LA protection, brute-force mathématiquement inutile face à une clé ED25519, et pas besoin de fermer le port 22.
- Une section **fail2ban** avec les lignes de commande en méthode nano : `sudo nano /etc/fail2ban/jail.local`, les 5 lignes à saisir, `Ctrl+O`/`Ctrl+X`, puis `sudo systemctl enable --now fail2ban`, et les commandes de vérification (`systemctl status`, `fail2ban-client status sshd`). Plus le piège à éviter : ne pas se tester soi-même avec 5 faux mots de passe.


> attention a faire attention au vrai trafic car si je restreint un nombre de requete par seconde je peux du coup restreindre un vrai traffic
