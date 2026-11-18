---
title: Flask dev server en prod pour un micro-service interne
tags: [python, infra, decision]
fiches-liees: [reseau-interne-docker, dockerfile-python, wiki-python, express-trust-proxy]
---

# Flask dev server en prod pour un micro-service interne

Stack : micro-service Python/Flask exposant PyMuPDF (traitement PDF), appelé uniquement par Node.js sur le réseau docker interne du VPS.
Usage : trancher si `python app.py` (serveur dev Flask) suffit en prod, et où se trouve le vrai goulot.

## Décision

**Pour un micro-service interne, `python app.py` passe très bien en prod**, tant que le conteneur n'est pas exposé directement à Internet — seul Node.js l'appelle, typiquement sur le réseau docker interne du VPS. Le serveur dev de Flask n'est pas le problème ; il ne faut juste pas le faire écouter au-delà du réseau interne (voir reseau-interne-docker.md).

## Le vrai point d'attention : la concurrence, pas le serveur HTTP

PyMuPDF est **CPU-bound**. Si Node envoie plusieurs PDF en parallèle, c'est le traitement PDF qui devient le goulot, pas le HTTP. Répartition :

- **Threads (`threaded=True`)** : quasi inutile pour du CPU-bound pur. Le GIL empêche deux threads d'exécuter du bytecode Python en parallèle ; les threads aident sur l'attente I/O, pas sur le calcul. Flask 1.0+ active d'ailleurs `threaded=True` par défaut — le préciser explicitement ne change rien.
- **Gunicorn avec quelques workers** : ce sont des **processus séparés**, chacun avec son interpréteur et son GIL → vraie parallélisation CPU. C'est le seul levier qui aide vraiment en parallèle.
- Ordre de grandeur : 2-4 workers suffisent pour un micro-service interne ; au-delà on se bat contre le CPU du VPS pour rien.

## Alléger si Flask pèse trop

Si Flask paraît trop lourd pour juste exposer PyMuPDF, on peut alléger : FastAPI (async, léger) ou même http.server pour les cas triviaux. **Ce n'est pas obligatoire** : le coût de Flask est négligeable ici, le changer n'améliore ni la latence ni le débit du traitement PDF.

## Anti-patterns

- Mettre gunicorn « pour faire propre » alors que la charge est séquentielle : complexité en plus, zéro gain mesurable.
- Compter sur `threaded=True` pour absorber du parallélisme PDF : ça ne résout pas le GIL.
- Exposer le conteneur Python directement sur Internet « parce que c'est un serveur dev » : la décision vaut pour un service interne, derrière Caddy/nginx, appelé par Node uniquement.

## État

**vivant** — décision validée 2026-11.
