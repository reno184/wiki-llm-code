# WebStorm AI Assistant — MCP GitHub (brancher wiki-llm-code)

Stack: WebStorm + JetBrains AI Assistant / Junie, protocole MCP, serveur GitHub MCP remote.
Usage: donner à l'agent de code de l'IDE un accès en lecture au wiki de patterns (repo GitHub `wiki-llm-code`) **sans cloner le repo dans le projet** (évite conflits git) et toujours à jour.

## Pourquoi MCP GitHub

- Le wiki vit sur GitHub (source de vérité). Le cloner à la racine du projet = conflit avec le repo git du projet.
- Le serveur GitHub MCP expose `get_file_contents` (et co.) : l'agent lit `INDEX.md` puis les fiches utiles à la demande, à jour.
- Vibe n'est **pas** un serveur MCP : ce qu'on branche, c'est GitHub, pas un assistant externe.

## Mise en place

### 1. Token GitHub — fine-grained (pas classic)

Developer settings → **Fine-grained token** :

- **Repository access** : *Only select repositories* → `wiki-llm-code` uniquement (repo public)
- **Permissions** : `Contents: Read-only`
- Expiration courte (≤ 90 jours)

Règle : un token = le moins de permissions possibles. Un token classic avec scope `repo` (full control private repos) est un excès — même fuite = zéro impact avec un fine-grained restreint. Copier le token immédiatement (plus réaffiché ensuite).

### 2. Config MCP dans WebStorm

Settings → Tools → **AI Assistant** → MCP → Add server :

```json
{
  "mcpServers": {
    "github": {
      "url": "https://api.githubcopilot.com/mcp/",
      "headers": {
        "Authorization": "Bearer ghp_TON_TOKEN"
      }
    }
  }
}
```

Points critiques :

- **La clé du serveur ne doit pas être vide** (`"github":`, pas `"":`) sinon la config est ignorée/cassée.
- **Project level plutôt que Global** : la config vit dans `.ai/mcp/mcp.json` (voir "Nouvelle initialisation projet" ci-dessous). ⚠️ Le token y est écrit en dur → le fichier est TOUJOURS dans le `.gitignore` (voir procédure).
- Global se justifie uniquement si le même serveur doit servir tous les projets JetBrains sans reconfig.


### 2b. Nouvelle initialisation projet (procédure systématique)

À chaque nouveau projet dans WebStorm :

**1. Ajouter le serveur via l'UI** (Settings → Tools → AI Assistant → MCP → Add server, Project level) OU créer directement le fichier `.ai/mcp/mcp.json` :

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

⚠️ L'UI peut créer le fichier **vide** (voir fiches d'expérience) : si le fichier existe mais est vide, le remplir à la main avec le contenu ci-dessus. Clé `"github"` non vide, JSON valide.

**2. Avant tout commit — vérifier le `.gitignore`** (le token est en dur dans le fichier) :

```
/.ai/mcp/mcp.json
```

- **Le slash initial est important** : il ancre le pattern à la racine du repo. `/.ai/mcp/mcp.json` ignore uniquement `<racine>/.ai/mcp/mcp.json` — sans le slash, git ignorerait aussi tout fichier de même nom dans un sous-répertoire (et surtout : un pattern non ancré reste ambigu).
- Si le fichier a déjà été committé : `git rm --cached .ai/mcp/mcp.json` puis commit — et comme le token a peut-être été exposé dans l'historique, **révoquer le token** et en générer un nouveau (fine-grained, Contents Read-only, ≤ 90 jours).

**3. Vérifier** : serveur `github` connecté dans l'UI MCP avec ses outils listés, et `git status` ne montre plus le fichier.

### 3. Vérification

Le serveur `github` doit apparaître connecté avec ses outils listés (`get_file_contents`…).

### 4. Premier prompt de test (à copier tel quel dans le chat AI Assistant)

```
Via le serveur MCP GitHub, lis le fichier INDEX.md du repo reno184/wiki-llm-code
(branche main), puis charge la fiche front/vuejs-router-menu.md.

Ensuite, décris-moi en 5 lignes comment tu ajouterais une page "DeleteMaterial"
à un projet Vue 3 selon ce pattern : liste exacte des fichiers à modifier,
avec ce qui change dans chacun. Ne modifie rien, juste le plan.
```

Pourquoi ce prompt est un bon test :

- **"Via le serveur MCP GitHub"** force l'agent à utiliser l'outil MCP (`get_file_contents`), pas une réponse générique.
- **INDEX.md d'abord, puis la fiche** : valide le workflow en 2 étapes conçu dans le wiki.
- **"Ne modifie rien, juste le plan"** : test sans risque, on vérifie que l'agent lit bien *notre* contenu avant de le laisser coder.

Critères de validation :

- ✅ La réponse contient des détails qui ne peuvent venir que de la fiche : `linkActiveClass: 'menu-active'`, entrée `menus[]` avec `short`, `:key="route.path"`, piège de l'unique élément racine.
- ❌ Réponse générique sans mention de `menu-active` → la fiche n'a pas été lue, MCP non utilisé.
- ❌ "Tool call refusé" / "pas d'accès à ce repo" → vérifier le header `Bearer <token>` et le périmètre du fine-grained token.

## Workflow agent : charger le wiki avant de coder

### 5. Règle persistante — `.junie/guidelines.md` (racine du projet)

```markdown
# Règles projet

Avant toute tâche de code, consulter le wiki de patterns via MCP GitHub :
repo reno184/wiki-llm-code → charger INDEX.md, puis uniquement les fiches utiles à la tâche.

- Ne pas réinventer : si une fiche couvre le besoin, suivre son code et ses pièges documentés.
- Toute déviation du pattern wiki doit être explicitement demandée par l'utilisateur, pas décidée seul.
```

Junie lit ce fichier à chaque session → plus besoin de répéter la consigne dans les prompts.

### 6. Prompt de tâche type

```
Ajoute une page DeleteMaterial. Entrée menu : label 'Delete material', short 'Delete', icône fa-trash.
Charge la fiche front/vuejs-router-menu.md et applique le pattern.
```

Les décisions documentées dans les fiches (ex. menu responsive) sont appliquées sans question ; les infos propres au projet (nom, label, icône) sont données dans le prompt ou demandées par l'agent.

## Notes

- MCP fonctionne avec les **modèles cloud** de JetBrains AI, pas avec les modèles locaux (limitation documentée JetBrains).
- Complétion de code (modèle Mellum) = illimitée et gratuite, ne consomme pas de crédits. Junie consomme des crédits : le réserver aux tâches multi-fichiers (voir discussions pricing AI Pro/Ultimate).
- Si le serveur refuse la connexion : vérifier le header `Authorization: Bearer <token>`, l'expiration du token et que le repo est bien dans le périmètre du fine-grained token.

## État

**vivant** — config réelle (WebStorm AI Assistant + GitHub MCP fine-grained token, repo wiki-llm-code).
