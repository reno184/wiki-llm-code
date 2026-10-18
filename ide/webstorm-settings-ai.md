# WebStorm Settings — AI Assistant (options à cocher)

Stack: WebStorm + JetBrains AI Assistant, Settings → Tools → AI Assistant.
Usage: décisions de configuration des options d'assistance IA de l'IDE — que cocher, que laisser décoché, et pourquoi (coût crédits, contrôle, contexte freelance client).

## Décisions (état validé 2026-10)

| Option | Décision | Pourquoi |
|---|---|---|
| Provide AI-generated name suggestions | ✅ cocher | Suggestions de noms (variables, fonctions, commits). Ne s'exécute pas tout seul, zéro inconvénient, gain de temps sur les renommages. |
| Suggest converting pasted code to the language of the target file | ✅ cocher | Convertit le code collé vers le langage du fichier (ex. Python → TypeScript). Rien d'automatique : simple suggestion refusable à chaque collage. Utile en fullstack (snippets doc/web). |
| Generate a title for the shelved changelist | ➖ indifférent | Ne concerne que les shelved changelists. Sans effet si on n'utilise pas cette feature. |
| Enable Insights — Generates brief explanations for complex or unclear code | ❌ ne pas cocher (sauf legacy inconnue) | Explications inline non demandées : polluent le scroll et redondantes avec "Explain" à la demande. Seul cas : reprise de codebase legacy inconnue, au niveau Project, à décocher ensuite. |
| GitHub Plugin: Generate a summary upon opening a Pull Request | ❌ ne pas cocher | Consomme des crédits à chaque ouverture de PR, même inutile. En contexte client freelance, la description de PR se rédige soi-même (ou à la demande via menu contextuel) — une description auto non relue fait bâclé. |

## Règles dérivées

- **Pas d'IA automatique sur ce qui est visible du client** (descriptions de PR, titres de commits poussés au remote) : générer à la demande, relire, reformuler.
- Les suggestions passives (noms, conversion de collage) sont OK : elles coûtent peu/rien et restent sous contrôle humain.
- Rappel crédits (voir fiche ide/webstorm-ai-mcp.md) : la complétion Mellum est illimitée/gratuite ; les features consommatrices (Junie, résumés) se réservent aux tâches multi-fichiers.

## Notes

- Emplacement exact : Settings → Tools → AI Assistant (section "Code" / onglet des options d'assistance).
- À re-réévaluer si JetBrains change le modèle de facturation ou ajoute des options.

## État

**vivant** — config réelle appliquée sur WebStorm (2026-10).
