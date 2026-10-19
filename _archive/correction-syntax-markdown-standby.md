---
name: correction-syntax-markdown-standby
layer: archive
version: 3
---

# Use case correction-syntax-markdown — archivé (stand-by)

**Archivé le 18/10/2026 : plus utilisé pour l'instant.** Réveil uniquement si un besoin réel de contrôle qualité Markdown apparaît dans un projet.

## Trace de la dernière version vivante

Passe de contrôle qualité sur un fichier Markdown entre pdf-to-md et anonymisation :

1. **Passe déterministe** (Prettier, `npx prettier --write doc.md`) : mise en forme, espaces, wrapping, listes, tableaux — ne reformule pas le texte.
2. **Passe IA locale** (Ollama) : hiérarchie des titres (sauts de niveau, H1 multiples, ponctuation de fin) — sortie toujours relue.

Historique des retraits : markdownlint retiré (non utilisé), puis mdformat retiré (non utilisé) — Prettier était resté la seule passe déterministe.

## Conséquences sur la chaîne

La chaîne documentaire devient directe : pdf-to-md → relecture humaine / anonymisation, **sans passe de contrôle qualité intermédiaire**. Prettier et Ollama restent documentés dans [libraries.md](../libraries.md) comme outils disponibles.
