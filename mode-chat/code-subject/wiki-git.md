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

## Ce que ça ne fait PAS

- `pull.rebase true` **n'interdit pas** un pull divergent : il le gère par rebase. Pour le refuser carrément (fast-forward strict) : `git config --global pull.ff only` — mais alors il faut choisir explicitement à chaque divergence, et les deux réglages combinés n'ont pas de sens.
- Ça ne change rien aux push rejetés (non-fast-forward) : là c'est `git push --force`… à ne jamais faire sur une branche partagée ; sur le wiki perso, `git pull --rebase` puis push suffit.

## Erreurs types et remèdes

| Situation | Cause | Remède |
|---|---|---|
| `cannot pull with rebase: Your index contains uncommitted changes` | modifs en cours, pas d'autoStash | `git stash` → `git pull --rebase` → `git stash pop` (ou activer autoStash) |
| Conflit pendant le rebase | même fichier touché local + distant | résoudre dans l'IDE → `git add <fichier>` → `git rebase --continue` (jamais un nouveau pull en plein rebase) |
| Envie d'abandonner un rebase en cours | conflit trop brouillon | `git rebase --abort` → retour à l'état d'avant |
| Repo d'équipe qui exige des merges explicites | convention du repo | dans CE repo : `git config pull.rebase false` (local, pas --global) |

## Niveaux de config

| Commande | Portée |
|---|---|
| `git config <clé> <val>` | le repo actuel (`.git/config`) |
| `git config --global` | tous les repos de l'utilisateur (`~/.gitconfig`) |
| `git config --system` | toute la machine, tous les utilisateurs (rare) |

## État

**vivant** — config appliquée sur le Mac de Renaud (oct. 2026) : `pull.rebase true` + `rebase.autoStash true`.
