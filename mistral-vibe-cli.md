---
title: Mistral Vibe CLI — agents custom pour réponse rapide
tags: [outils, piege]
fiches-liees: [webstorm-agents-junie-vibe, webstorm-settings-ai]
---

# Mistral Vibe CLI — agents custom pour réponse rapide

Stack : Vibe CLI (mistral-vibe) en terminal, macOS.
Usage : rendre le CLI utilisable pour un simple avis sur un fichier, au lieu de 3 minutes d'exploration agentique.

## Piège de départ

Demander « ton avis sur ce fichier de 10 lignes » dans le CLI = 3 minutes d'attente. Cause : modèle de raisonnement lourd par défaut + boucle d'outils (exploration du projet, commandes) même quand la question ne le justifie pas. Il n'y a pas de « mode non-agent » : tout passe par le système d'agents. La solution est un agent custom minimal, pas un réglage caché.

## L'agent custom « avis rapide »

Le CLI charge les agents depuis `~/.vibe/agents/<nom>.toml` (utilisateur, tous projets) ou `./.vibe/agents/<nom>.toml` (projet). Le dossier `agents/` n'existe pas par défaut, il faut le créer.

Création macOS, en une commande (terminal zsh) :

```bash
mkdir -p ~/.vibe/agents && cat > ~/.vibe/agents/avis.toml << 'EOF'
display_name = "Avis rapide"
description = "Répond directement, sans explorer ni modifier le projet."
safety = "safe"
enabled_tools = ["read_file"]
instructions = """
Réponds directement à la question sur le fichier fourni.
N'explore pas le projet, ne lance pas de commandes, ne propose pas de modifications.
"""
EOF
```

Utilisation :

- `vibe --agent avis` au lancement, ou `default_agent = "avis"` dans `~/.vibe/config.toml`.
- `Shift+Tab` pour basculer d'agent à la volée dans la session (ask → accept-edits → Avis rapide…).
- `display_name` ne s'affiche pas en grand : c'est le libellé du mode près de la zone de saisie, visible dans la rotation Shift+Tab.
- Pour un avis encore plus rapide : coller le contenu du fichier directement dans le prompt plutôt que de viser son chemin (zéro exploration).

## Piège : active_model avec un nom invalide

Mettre `active_model = "mistral-small-3.2"` (nom catalogue, pas nom API) dans le TOML peut empêcher le chargement de l'agent : `vibe --agent avis` démarre alors sans l'agent, sans libellé « Avis rapide » dans la rotation. Correct : soit retirer la ligne (l'agent suit le modèle de la session, changeable avec `/model`), soit mettre un nom API valide vu dans `/model`.

## Dépannage rapide

- Vérifier le fichier : `cat ~/.vibe/agents/avis.toml` (contenu complet, pas de backslashes parasites).
- Les warnings affichés au démarrage de la session (agent not found, unknown model) disent tout : les lire avant de chercher ailleurs.

## État

**vivant** — validé 2026-11 sur Mac M4.
