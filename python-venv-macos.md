---
title: Lancer Python sur Mac M4 — venv vs Python système (serveur Flask en ligne de commande)
tags: [python, outils, howto]
fiches-liees: [wiki-python, webstorm-gitignore-run-configurations, checklist-initialize-project]
---

# python-venv

## Le rôle de chaque chose

- **Python système** (`/usr/bin/python3`, 3.9.6, fourni par macOS). Il sert de base pour créer des environnements, pas pour développer.
- **Environnement virtuel (venv)** : un dossier `.venv/` avec son propre jeu de paquets pip, isolé du système et des autres projets. Il hérite de la version du Python utilisé pour le créer (3.12 ici).

## Lancer l'API — la recette

WebStorm : la flèche verte ▶ et le reste ne fonctionne pas pour Python.

```bash
# une fois par projet, depuis le dossier de l'API
cd /Users/<user>/Code/<projet>
/opt/homebrew/bin/python3.12 -m venv .venv       # Homebrew = Apple Silicon natif, version upgradable
source .venv/bin/activate
pip install -r requirements.txt
deactivate

# ou avec activation (python/pip tapés courts)
source .venv/bin/activate
python app.py
deactivate                      # sortir du venv
```

## Mettre à jour les dépendances

- Après un `pip install <nouvelle-lib>` : `pip freeze > requirements.txt` pour resynchroniser le fichier.
- `rm -rf .venv` puis recréation : seulement pour changer de version de Python ou repartir propre (les paquets se réinstallent depuis `requirements.txt`).

## Snippet de lancement (app.py)

```python
import os

if __name__ == "__main__":
    debug_mode = os.getenv("FLASK_DEBUG", "1") == "1"
    app.run(host="127.0.0.1", port=5000, debug=debug_mode)
```

- `FLASK_DEBUG=1` (défaut) : recharge à chaque sauvegarde.
- `host="127.0.0.1"` à la place de `0.0.0.0` car écoute sur toutes les interfaces pas bon.

## Pièges

- Utiliser `python3` du PATH pour créer le venv → on hérite du 3.9.6 figé de macOS. Toujours `/opt/homebrew/bin/python3.12` explicite.


## Liens croisés

- [wiki-python.md](wiki-python.md) — déboguer ce serveur Flask (points d'arrêt, prints `flush=True`) avec PyCharm ou VS Code.
- [webstorm-gitignore-run-configurations.md](webstorm-gitignore-run-configurations.md) — gérer le `.gitignore` du projet.
- [checklist-initialize-project.md](checklist-initialize-project.md) — checklist d'initialisation projet, le venv en fait partie côté Python.
