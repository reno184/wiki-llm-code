---
title: Fermer les ports / firewall
tags: [securite, decision]
---
# Fermer le port 22 quand tu ne l'utilises pas : fausse bonne idée

## Pourquoi c'est une fausse bonne idée

- Il faut le rouvrir à chaque utilisation → tu finiras par le laisser ouvert, ou tu le feras moins souvent que nécessaire.
- Ton IP box change régulièrement (sauf IP fixe) → si tu restreins à ton IP, tu peux te verrouiller dehors (sauf à passer par la console Infomaniak, qui permet toujours de rouvrir — c'est ta planche de salut).
- Ça protège seulement les heures de fermeture, alors que la protection n°1 (auth par clé, `PasswordAuthentication no`) protège tout le temps.

## Exception où fermer le port a du sens

Un serveur que tu n'administres que rarement (ex. une fois par mois), où tu rouvres via le pare-feu Infomaniak juste le temps de ta session. Pour un VPS de travail quotidien, c'est trop de friction.
