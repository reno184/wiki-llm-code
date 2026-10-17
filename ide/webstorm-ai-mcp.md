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
- **Project level plutôt que Global** : la config (fichier `.mcp.json` à la racine) est versionnée avec le projet, retrouvée sur toutes les machines. ⚠️ Si le token est écrit en dur dans `.mcp.json`, ajouter le fichier au `.gitignore` (ou laisser l'UI WebStorm le stocker hors repo).
- Global se justifie uniquement si le même serveur doit servir tous les projets JetBrains sans reconfig.

### 3. Vérification

Le serveur `github` doit apparaître connecté avec ses outils listés (`get_file_contents`…). Test dans le chat :

> *"Via MCP GitHub, lis INDEX.md du repo reno184/wiki-llm-code et résume les fiches front."*

## Workflow agent : charger le wiki avant de coder

### 4. Règle persistante — `.junie/guidelines.md` (racine du projet)

```markdown
# Règles projet

Avant toute tâche de code, consulter le wiki de patterns via MCP GitHub :
repo reno184/wiki-llm-code → charger INDEX.md, puis uniquement les fiches utiles à la tâche.

- Ne pas réinventer : si une fiche couvre le besoin, suivre son code et ses pièges documentés.
- Toute déviation du pattern wiki doit être explicitement demandée par l'utilisateur, pas décidée seul.
```

Junie lit ce fichier à chaque session → plus besoin de répéter la consigne dans les prompts.

### 5. Prompt de tâche type

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
