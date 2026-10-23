# Checklist — initialiser un projet avec agents + MCP GitHub (WebStorm)

Stack: WebStorm (AI Assistant / Junie), MCP GitHub remote.

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


**2. Ne pas oublier gitignore, ne pas ajouter il y une clef dedans !**  :

```
/.ai/mcp/mcp.json
```

**3. Rules projet — `.aiassistant/rules/`** : finalement je n'utilise pas je préfère .guidelines dans le dossier .junie, au moins je suis sur qu'il n'y a que cette agent qui l'utilise

> bien relire guidelines.md avant de récoder

## État

**vivant**
