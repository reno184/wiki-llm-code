---
title: Lancer Python sur Mac M4 — venv vs Python système (serveur Flask en ligne de commande)
tags: [python, outils, howto]
fiches-liees: [wiki-python, webstorm-gitignore-run-configurations, checklist-initialize-project]
---

# python-venv-macos — Lancer un projet Python sur Mac M4 : venv vs Python système

Contexte : un projet a un `requirements.txt` (Flask, PyMuPDF, lxml, Pillow). Symptôme de départ : `/usr/bin/python3` ne trouve pas Flask (`ModuleNotFoundError: No module named 'flask'`). Ce n'est pas un bug : le Python système n'a jamais reçu ces paquets, et il ne faut pas les y installer.

## Le rôle de chaque chose

- **Python système** (`/usr/bin/python3`, 3.9.6, fourni par macOS) : l'interpréteur à nu, sans librairie tierce, version figée. Il sert de base pour créer des environnements, pas pour développer.
- **Environnement virtuel (venv)** : un dossier `.venv/` avec son propre jeu de paquets pip, isolé du système et des autres projets. Il hérite de la version du Python utilisé pour le créer (3.12 ici).
- **Pourquoi pas en global** : conflits de versions entre projets, système pollué, et pip refuse carrément (`externally-managed-environment`, PEP 668) sur les Python système/Homebrew. Le venv est le chemin standard, pas un raffinement d'équipe.

## Lancer l'API — la recette

WebStorm : la flèche verte ▶ ne fonctionne pas pour Python dans ce projet. On lance depuis un terminal, depuis le dossier de l'API Python.

```bash
# une fois par projet, depuis le dossier de l'API
cd /Users/<user>/Code/<projet>
/opt/homebrew/bin/python3.12 -m venv .venv       # Homebrew = Apple Silicon natif, version upgradable
.venv/bin/pip install -r requirements.txt

# chaque session : lancer le serveur (port 5000)
cd /Users/<user>/Code/<projet>
.venv/bin/python app.py          # Ctrl+C pour arrêter

# ou avec activation (python/pip tapés courts)
source .venv/bin/activate
python app.py
deactivate                      # sortir du venv
```

Le serveur répond sur http://127.0.0.1:5000/ — vérifier au navigateur ou `curl http://127.0.0.1:5000/`. Le lendemain, un simple `source .venv/bin/activate` (ou `.venv/bin/python app.py`) suffit, rien à réinstaller.

## Mettre à jour les dépendances

- Une lib ajoutée au `requirements.txt` : `.venv/bin/pip install -r requirements.txt` (ou `pip install -r requirements.txt` venv activé).
- Après un `pip install <nouvelle-lib>` : `pip freeze > requirements.txt` pour resynchroniser le fichier.
- `rm -rf .venv` puis recréation : seulement pour changer de version de Python ou repartir propre (les paquets se réinstallent depuis `requirements.txt`).

## Snippet de lancement (app.py)

```python
import os

if __name__ == "__main__":
    debug_mode = os.getenv("FLASK_DEBUG", "1") == "1"
    app.run(host="127.0.0.1", port=5000, debug=debug_mode)
```

- `FLASK_DEBUG=1` (défaut) : recharge à chaque sauvegarde + page d'erreur interactive. `FLASK_DEBUG=0 python app.py` pour couper.
- `host="127.0.0.1"` en local. `0.0.0.0` écoute sur toutes les interfaces : avec debug=1, le debugger interactif Flask (exécution de code arbitraire) est exposé au réseau — `0.0.0.0` uniquement en déploiement derrière reverse-proxy.

## Pièges

- Lancer `python app.py` venv non activé → `ModuleNotFoundError` identique au bug initial. Trancher en une seconde : `which python` doit pointer dans `.venv/bin/`.
- Utiliser `python3` du PATH pour créer le venv → on hérite du 3.9.6 figé de macOS. Toujours `/opt/homebrew/bin/python3.12` explicite.
- Flask, PyMuPDF, lxml, Pillow ont tous des wheels Apple Silicon : l'install est rapide, sans compilation. Si pip compile, vérifier qu'on n'est pas sur un Python Intel sous Rosetta (`file $(which python3)`).

## Checklist

- [ ] `/opt/homebrew/bin/python3.12 -m venv .venv` à la racine de l'API
- [ ] `.venv/bin/pip install -r requirements.txt`
- [ ] Lancer avec `.venv/bin/python app.py` ou venv activé (`which python` dans `.venv/bin/`)
- [ ] `.venv/` dans le `.gitignore`

## Liens croisés

- [wiki-python.md](wiki-python.md) — déboguer ce serveur Flask (points d'arrêt, prints `flush=True`) avec PyCharm ou VS Code.
- [webstorm-gitignore-run-configurations.md](webstorm-gitignore-run-configurations.md) — gérer le `.gitignore` du projet.
- [checklist-initialize-project.md](checklist-initialize-project.md) — checklist d'initialisation projet, le venv en fait partie côté Python.
