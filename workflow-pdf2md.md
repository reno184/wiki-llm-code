1. **Passe déterministe** (Prettier, `npx prettier --write doc.md`) : mise en forme, espaces, wrapping, listes, tableaux — ne reformule pas le texte.
2. **Passe IA locale** (Ollama) : hiérarchie des titres (sauts de niveau, H1 multiples, ponctuation de fin) — sortie toujours relue.

Exemple utiliser python avec get_table

page.find_tables() te renvoie les tableaux avec leurs cellules : reconstruire un tableau Markdown est trivial (join de |). Tu peux même le faire en pur Python sans l'IA, et demander à l'IA seulement de vérifier/nettoyer.
