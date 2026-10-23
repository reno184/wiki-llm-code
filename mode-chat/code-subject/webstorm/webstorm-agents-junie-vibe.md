# WebStorm agents — Junie vs Mistral Vibe (stratégie crédits)

Stack: WebStorm 2025.3.2+ + AI Assistant plugin, agent Mistral Vibe via ACP, abonnement JetBrains + Mistral Pro.
Usage: répartition des tâches IA entre Junie (crédits JetBrains limités) et Vibe (abonnement Mistral Pro) pour ne jamais tomber à court de crédits.

## Principe du billing

- **Junie / AI Assistant JetBrains** → consomme les crédits JetBrains (offre incluse : ~30/mois).
- **Mistral Vibe (agent ACP)** → consomme l'abonnement **Mistral Pro**, pas les crédits JetBrains. Même moteur, agents, skills et MCP que Vibe ailleurs.
- **Complétion inline (Mellum)** → illimitée, 0 crédit. Toujours active.

## Installation de Vibe dans WebStorm

Settings → Tools → AI Assistant → **Agents** → Search agents → "Mistral Vibe" → installer et connecter (compte Mistral). Exigé : IDE 2025.3.2+ (annuaire d'agents ACP) + AI Assistant plugin. Une fois connecté, le chat natif de l'IDE tourne contre l'agent Vibe avec le contexte du projet ouvert.

⚠️ Ne pas confondre avec le plugin "Mistral Code Enterprise" (usage entreprise, pas ce cas).

## Répartition des tâches

| Tâche | Agent | Pourquoi |
|---|---|---|
| Tâches multi-fichiers, refactoring, génération de code | Vibe | Facturé Mistral Pro, limites plus larges que 30 crédits/mois |
| Complétions inline | Mellum | Illimité, 0 crédit |
| Intégration IDE profonde (run configs, inspections, refactor natif) | Junie | Dosé : c'est le seul cas qui vaut des crédits JetBrains |
| Découverte codebase legacy | Junie ponctuel, sinon Vibe | Vibe par défaut, Junie si l'agent doit exécuter/débugger dans l'IDE |

Règle : **Junie = outil ciblé, Vibe = agent par défaut.** Le réflexe avant chaque tâche : est-ce que ça a besoin de l'IDE JetBrains ? Non → Vibe.

## Notes

- Vibe agent lit les mêmes MCP que l'IDE (le serveur GitHub du wiki, voir fiche ide/webstorm-ai-mcp.md, reste utilisable).
- Si les crédits JetBrains passent en payant ou que l'offre change, re-réévaluer la répartition.

## État

**vivant** — stratégie validée 2026-10.
