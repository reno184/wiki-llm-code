---
title: Lancer Python sur Mac M4 — venv vs Python système (serveur Flask dans WebStorm)
tags: [python, outils, howto]
fiches-liees: [wiki-python, webstorm-gitignore-run-configurations, checklist-initialize-project]
---

# python-venv-macos — Lancer un projet Python sur Mac M4 : venv vs Python système

Contexte : un projet a un `requirements.txt` (Flask, PyMuPDF, lxml, Pillow) mais aucun environnement virtuel. Symptôme : `/usr/bin/python3` ne trouve pas Flask (`ModuleNotFoundError: No module named 'flask'`). Ce n'est pas un bug, c'est le comportement normal : le Python du système n'a jamais reçu ces paquets, et il ne faut pas les y installer.

## Le rôle de chaque chose

- **Python installé en local (le "système")** : l'interpréteur seul, à nu. Sur macOS c'est `/usr/bin/python3` (stub fourni par les Command Line Tools Xcode). Il ne connaît que la bibliothèque standard. Il sert de **base** pour créer des environnements, pas pour开发 directement.
- **Environnement virtuel (venv)** : un dossier (`.venv/`) copiant le nécessaire pour tourner avec son **propre jeu de paquets pip**, isolé du système et des autres projets. Chaque projet a le sien ; les versions de Flask/lxml d'un projet n'écrasent jamais celles d'un autre.
- **Pourquoi ne pas installer les paquets en global** : conflits de versions entre projets, système pollué, et depuis quelques années pip le **refuse carrément** sur les Python Homebrew/système (`error: externally-managed-environment`, PEP 668). Le venv n'est pas un raffinement d'équipe, c'est le chemin de moindre effort.

## Recette (une fois par projet)

```bash
# à la racine du projet
python3 -m venv .venv

# activer (macOS/zsh)
source .venv/bin/activate   # le prompt affiche (.venv)

# installer les dépendances du projet
pip install -r requirements.txt
```

Une fois activé, `python` et `pip` pointent vers le venv. Lancer le serveur :

```bash
python app.py            # ou: flask run, ou: python -m flask --app app run
```

Sortir du venv : `deactivate`. Le lendemain, un simple `source .venv/bin/activate` suffit, rien à réinstaller.

## Dans WebStorm (Mac M4)

- **Terminal intégré** : `View → Tool Windows → Terminal`. Si le venv n'est pas activé automatiquement, `source .venv/bin/activate` dans ce terminal, puis lancer le serveur comme ci-dessus (utile pour voir les prints `flush=True` en direct).
- WebStorm **ne peut pas déboguer Python** (plugin réservé à PyCharm / IntelliJ Ultimate) mais il lance et lit les logs sans problème via le terminal. Voir la fiche dédiée au débogage.
- Ajouter `.venv/` au `.gitignore` du projet.

## Pièges Mac M4

- **Oublier d'activer le venv** avant `pip install` ou `python app.py` → erreurs `ModuleNotFoundError` identiques au bug initial. Vérifier : `which python` doit pointer dans `.venv/bin/`.
- **Les wheelers arm64** : Flask, PyMuPDF, lxml et Pillow fournissent tous des wheels natifs Apple Silicon, `pip install -r requirements.txt` s installe en quelques secondes sans compilation. Si pip veut compiler, vérifier qu'on n'est pas sur un Python Intel sous Rosetta (`file $(which python3)`).
- **`/usr/bin/python3` = version unique et figée** par macOS : ne jamais y toucher pour les projets, uniquement pour créer les venv. Si besoin d'une autre version : `brew install python@3.12` puis créer le venv avec ce binaire.

## Checklist

- [ ] `python3 -m venv .venv` à la racine
- [ ] `source .venv/bin/activate` avant toute commande pip/python
- [ ] `pip install -r requirements.txt`
- [ ] `.venv/` dans le `.gitignore`
- [ ] `which python` pointe bien dans `.venv/bin/`

## Liens croisés

- [wiki-python.md](wiki-python.md) — déboguer ce serveur Flask (points d'arrêt, prints `flush=True`), car WebStorm ne débogue pas le Python.
- [webstorm-gitignore-run-configurations.md](webstorm-gitignore-run-configurations.md) — gérer le `.gitignore` et les run configurations versionnées.
- [checklist-initialize-project.md](checklist-initialize-project.md) — checklist d'initialisation projet, le venv en fait partie côté Python.
