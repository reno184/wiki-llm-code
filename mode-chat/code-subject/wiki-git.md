# wiki-git — pull.rebase true : historique linéaire, plus de merge commit

Usage: configuration Git de la machine de Renaud (appliquée globalement, `~/.gitconfig`). À relire en cas d'historique tordu ou de conflit au pull. Jamais chargé par un agent.

## La commande

```bash
git config --global pull.rebase true
```

`--global` = toute la machine, tous les repos de l'utilisateur (`~/.gitconfig`), pas un seul projet. Réversible : `git config --global --unset pull.rebase`.

## Ce que ça change

Sans config, `git pull` sur une divergence (commit local + commit distant) crée un **commit de merge** — un nœud à deux parents qui pollue l'historique. Avec `pull.rebase true` :

- Fast-forward quand c'est possible (branche locale juste en retard) → rien de visible.
- Divergence → `git pull` fait un **rebase automatique** : les commits locaux sont rejoués par-dessus `origin/main`. Historique linéaire, zéro commit de merge.

## Compagnon indispensable : autoStash

```bash
git config --global rebase.autoStash true
```

Sans lui, un rebase refuse de démarrer si des modifs sont en cours (`error: cannot pull with rebase: Your index contains uncommitted changes`). Avec lui : stash automatique avant le rebase, unstash après — l'erreur ne peut plus se produire.

## supprimer le cache

- Si déjà committé : `git rm --cached ./xx/xx.json` 


## État

**vivant** — config appliquée sur le Mac de Renaud (oct. 2026) : `pull.rebase true` + `rebase.autoStash true`.
