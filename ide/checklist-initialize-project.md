# Checklist — initialiser un projet avec agents + MCP GitHub (WebStorm)

Stack: WebStorm (AI Assistant / Junie), MCP GitHub remote, repo wiki-llm-code.
Usage: **checklist systématique à chaque nouveau projet** — brancher les agents IDE sur le wiki via MCP, sans jamais exposer le token. La source de vérité du wiki est le repo GitHub [reno184/wiki-llm-code](https://github.com/reno184/wiki-llm-code) ; on ne le clone pas dans le projet (conflit git).

## Prérequis — token GitHub fine-grained

- Developer settings → **Fine-grained token** (pas classic) : *Only select repositories* → `wiki-llm-code`, permissions `Contents: Read-only`, expiration ≤ 90 jours.

## Checklist

**1. Ajouter le serveur MCP** (Settings → Tools → AI Assistant → MCP → Add server, **Project level**) ou créer directement `.ai/mcp/mcp.json` :

```json
{
  "mcpServers": {
    "github": {
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer MON-TOKEN"
      }
    }
  }
}
```

⚠️ Piège : l'UI peut créer le fichier **vide** → si le fichier existe mais est vide, le remplir à la main. Clé `"github"` non vide, JSON valide.

**2. Avant tout commit — gitignore du fichier token** (le token est en dur dedans) :

```
/.ai/mcp/mcp.json
```

- **Le slash initial est important** : il ancre le pattern à la racine du repo, sinon git ignorerait tout fichier de même nom dans un sous-répertoire.
- Si déjà committé : `git rm --cached .ai/mcp/mcp.json` + commit, puis **révoquer et régénérer le token** (il a pu rester dans l'historique).

**3. Rules projet — `.aiassistant/rules/`** (racine du projet), 3 fichiers :

`agent-code-first.md` — routage wiki pour coder :

```markdown
- Connecte-toi au serveur MCP GitHub (déjà configuré dans ce projet).
- Charge README.md (règles globales) puis INDEX.md du repo reno184/wiki-llm-code.
- Si AUCUN profil n'est indiqué dans le prompt actuel, demande-moi lequel je
  suis AVANT de charger le moindre fichier (économie de tokens). Exemples :
  developpeur-javascript, python-docker.
- Dans INDEX.md : cherche le profil indiqué dans le prompt actuel ; si
  introuvable, arrête immédiatement et pose-moi la question.
- Ne regarde que les fiches liées au profil.
- Ne jamais proposer de structure alternative qui contredit une fiche du wiki.
```

`agent-code-readable.md` (frontmatter `apply: always`) — style prototype lisible :

```markdown
---
apply: always
---

- Mode par défaut : prototype en développement personnel. Écrire le code
  pour ÊTRE LU, pas pour être blindé.
- Pas de programmation défensive : les paramètres reçus sont censés être
  corrects, la base de données est censée renvoyer le record attendu.
- Pas de gestion d'erreur cosmétique : intercepter une erreur uniquement si
  on sait quoi en faire (retry, message utile, fallback réel) — sinon
  laisser remonter.
- La lisibilité prime : une action par étape, lignes vides entre blocs,
  lecture verticale sans sauter.
- La répétition est tolérée : répéter 3 lignes deux fois va mieux qu'une
  abstraction prématurée. Pas d'utilitaire tant que le besoin n'est pas
  répété et stabilisé.
- Zéro commentaire dans le code généré (nommage auto-documentant) — deux
  exceptions uniquement : 1. workaround non devinable (piège,
  décision contre-intuitive). 2. Méthode de librairie tierce dont le
  comportement n'est pas devinable depuis le nom (ex. Python : modification
  en place, stream consommé).
```

`chat-github-mcp.md` — version chat pur (consultation SANS regarder le projet local — leçon Gemini Flash/requirements.txt : un petit modèle charge le répertoire de travail au lieu du wiki) :

```markdown
- Connecte-toi au serveur MCP GitHub (déjà configuré dans ce projet).
- Charge README.md puis INDEX.md du repo reno184/wiki-llm-code.
- Si AUCUN profil n'est indiqué dans le prompt, demande-moi lequel AVANT
  de charger le moindre fichier.
- Il t'est INTERDIT de regarder le répertoire de travail local :
  réponse basée uniquement sur le contenu du wiki.
```

**4. Règle persistante Junie — `.junie/guidelines.md`** (racine du projet) :

```markdown
# Règles projet

Avant toute tâche de code, consulter le wiki de patterns via MCP GitHub :
repo reno184/wiki-llm-code → charger INDEX.md, puis uniquement les fiches utiles à la tâche.

- Ne pas réinventer : si une fiche couvre le besoin, suivre son code et ses pièges documentés.
- Toute déviation du pattern wiki doit être explicitement demandée par l'utilisateur, pas décidée seul.
```

**5. Vérifier** : serveur `github` connecté avec ses outils listés (`get_file_contents`…), et `git status` ne montre plus `.ai/mcp/mcp.json`.

**6. Premier prompt de test** (à copier tel quel) :

```
Via le serveur MCP GitHub, lis le fichier INDEX.md du repo reno184/wiki-llm-code
(branche main), puis charge la fiche front/vuejs-router-menu.md.

Ensuite, décris-moi en 5 lignes comment tu ajouterais une page "DeleteMaterial"
à un projet Vue 3 selon ce pattern : liste exacte des fichiers à modifier,
avec ce qui change dans chacun. Ne modifie rien, juste le plan.
```

- ✅ Réponse contenant des détails qui ne peuvent venir que de la fiche (`menu-active`, entrée `menus[]` avec `short`, `:key="route.path"`, piège de l'unique racine) → MCP et wiki OK.
- ❌ Réponse générique sans mention de `menu-active` → fiche non lue, MCP non utilisé.
- ❌ "Tool call refusé" → vérifier le header `Bearer <token>`, l'expiration et le périmètre du fine-grained token.

## Prompt de tâche type (après init)

```
Ajoute une page DeleteMaterial. Entrée menu : label 'Delete material', short 'Delete', icône fa-trash.
Charge la fiche front/vuejs-router-menu.md et applique le pattern.
```

Les décisions des fiches sont appliquées sans question ; les infos propres au projet (nom, label, icône) sont données dans le prompt ou demandées par l'agent.

## Notes

- MCP = modèles cloud JetBrains uniquement (pas les modèles locaux).
- Mellum (complétion) illimité/gratuit ; Junie consomme des crédits → réserver aux tâches multi-fichiers.
- Profil à indiquer dans le prompt : voir la section Profils de l'INDEX du wiki.

## État

**vivant** — checklist consolidée depuis la config réelle (ex-webstorm-ai-mcp, 19/10/2026) ; valable pour tout nouveau projet WebStorm avec agents.
