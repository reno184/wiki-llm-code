---
title: Lancer Python sur Mac M4 — venv vs Python système (serveur Flask dans WebStorm)
tags: [python, outils, howto]
fiches-liees: [wiki-python, webstorm-gitignore-run-configurations, checklist-initialize-project]
---

# python-venv-macos — Lancer un projet Python sur Mac M4 : venv vs Python système

Contexte : un projet a un `requirements.txt` (Flask, PyMuPDF, lxml, Pillow) mais aucun environnement virtuel. Symptôme : `/usr/bin/python3` ne trouve pas Flask (`ModuleNotFoundError: No module named 'flask'`). Ce n'est pas un bug, c'est le comportement normal : le Python du système n'a jamais reçu ces paquets, et il ne faut pas les y installer.

## Le rôle de chaque chose

- **Python installé en local (le "système")** : l'interpréteur seul, à nu. Sur macOS c'est `/usr/bin/python3` (stub fourni par les Command Line Tools Xcode). Il ne connaît que la bibliothèque standard. Il sert de **base** pour créer des environnements, pas pour développer directement.
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

## Workflow 100 % ligne de commande (sans IDE Python)

Aucun IDE Python nécessaire : le venv gère tout depuis le terminal.

```bash
# une fois par projet
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

# chaque session
cd <projet>
source .venv/bin/activate
python app.py        # Ctrl+C pour arrêter
deactivate           # sortir du venv
```

Lancement typique d'un serveur Flask (`app.py`) :

```python
import os

if __name__ == "__main__":
    debug_mode = os.getenv("FLASK_DEBUG", "1") == "1"
    app.run(host="127.0.0.1", port=5000, debug=debug_mode)
```

- `FLASK_DEBUG=1` (défaut ici) : recharge à chaque sauvegarde + page d'erreur interactive. `FLASK_DEBUG=0 python app.py` pour couper.
- `host="127.0.0.1"` en local. `0.0.0.0` écoute sur toutes les interfaces : avec debug=1, le debugger interactif Flask (qui permet l'exécution de code arbitraire) est exposé au réseau. `0.0.0.0` uniquement en déploiement derrière reverse-proxy.
- Vérifier au doute : `which python` doit pointer dans `.venv/bin/`.

## Choisir l'interpréteur dans WebStorm (piège de la liste "Python 3.9")

WebStorm propose une liste d'interpréteurs Python : le système (`/usr/bin/python3`, souvent 3.9 sur macOS), les installs Homebrew, et les venv du projet. **Le venv n'est pas une alternative à l'interpréteur : il EST l'un des interpréteurs de la liste.** Il apparaît comme `.venv/bin/python`, souvent affiché « Python 3.x (nom-du-projet) ».

- Choisir la ligne « 3.9 » toute seule = le Python système à nu = `ModuleNotFoundError: No module named 'flask'` (il n'a jamais les paquets du requirements.txt).
- Choisir la ligne du venv = l'environnement avec Flask, PyMuPDF, lxml, Pillow installés.
- Vérification imparable : la commande de run affichée doit être `<projet>/.venv/bin/python <projet>/app.py`. Si le chemin ne contient pas `.venv/bin/python`, mauvais interpréteur.
- L'app est ensuite servie sur http://127.0.0.1:5000/.

## Lancer / lire les logs dans WebStorm (Mac M4)

- **Terminal intégré** : `View → Tool Windows → Terminal`. `source .venv/bin/activate` si le venv n'est pas activé automatiquement, puis lancer le serveur (utile pour voir les prints `flush=True` en direct).
- Flèche verte ▶ à côté de `app.py` une fois le bon interpréteur sélectionné.
- Le débogage avancé (points d'arrêt) reste PyCharm / VS Code : voir la fiche dédiée.
- Ajouter `.venv/` au `.gitignore` du projet.

## Pièges Mac M4

- **Oublier d'activer le venv** avant `pip install` ou `python app.py` → erreurs `ModuleNotFoundError` identiques au bug initial. Vérifier : `which python` doit pointer dans `.venv/bin/`.
- **D'où vient ce « Python 3.9 »** : `/usr/bin/python3` est le Python 3.9.6 embarqué avec macOS (CommandLineTools), sans aucune librairie tierce. Le venv créé avec lui hérite de sa version (3.9) tout en ayant ses propres paquets depuis `requirements.txt`. Le système reste intact : c'est le comportement voulu, chaque projet gère ses dépendances isolément.
- **Les wheels arm64** : Flask, PyMuPDF, lxml et Pillow fournissent tous des wheels natifs Apple Silicon, `pip install -r requirements.txt` s'installe en quelques secondes sans compilation. Si pip veut compiler, vérifier qu'on n'est pas sur un Python Intel sous Rosetta (`file $(which python3)`).
- **`/usr/bin/python3` = version unique et figée** par macOS : ne jamais y toucher pour les projets, uniquement pour créer les venv. Si besoin d'une autre version : `brew install python@3.12` puis créer le venv avec ce binaire.

## Mettre à jour les dépendances du projet

Si une librairie est ajoutée au `requirements.txt`, la réinstaller dans le venv :

```bash
# depuis un terminal où le venv est activé
pip install -r requirements.txt

# sans activer le venv, en appelant pip par son chemin complet
/Users/<user>/Code/<projet>/.venv/bin/pip install -r requirements.txt
```

Le chemin complet `.venv/bin/pip` marche sans activation : pratique dans les scripts/CI. `pip freeze > requirements.txt` pour mettre à jour le fichier après un `pip install <nouvelle-lib>`.

## Checklist

- [ ] `python3 -m venv .venv` à la racine
- [ ] `source .venv/bin/activate` avant toute commande pip/python
- [ ] `pip install -r requirements.txt`
- [ ] `.venv/` dans le `.gitignore`
- [ ] `which python` pointe bien dans `.venv/bin/`

## Liens croisés

- [wiki-python.md](wiki-python.md) — déboguer ce serveur Flask (points d'arrêt, prints `flush=True`), le débogage Python reste PyCharm / VS Code.
- [webstorm-gitignore-run-configurations.md](webstorm-gitignore-run-configurations.md) — gérer le `.gitignore` et les run configurations versionnées.
- [checklist-initialize-project.md](checklist-initialize-project.md) — checklist d'initialisation projet, le venv en fait partie côté Python.
