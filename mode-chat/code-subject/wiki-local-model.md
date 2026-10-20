# wiki-local-model — modèles locaux : inventaire, paramètres, limites

Usage: référence unique sur les modèles Ollama utilisés par Renaud (jamais chargé par un agent). Runtime et patterns → [wiki-ollama.md](wiki-ollama.md).

## Modèle vision de référence : frob/qwen3.5-instruct:9b

- **Besoin couvert** : OCR / extraction d'informations depuis une image (PNG capture, scan, photo). ~9B paramètres — tourne sur machine locale via CLI desktop Ollama.
- **Tests** (oct. 2026) :
  - Fiche de lecture Harry Potter (PNG) → convenable, SAUF liste de personnages en fin de fiche (interligne serré : 3 ratés) → contournement validé : capture ciblée sur la zone dense = résultat nickel.
  - Fausse note manuscrite médicale (PNG généré par IA) → **parfait, réussite absolue**, manuscrit lu intégralement.
- **Périmètre de validité** :
  - ✅ **Extraction courte vision** (1-2 pages, imprimé ou manuscrit) — moteur principal de la piste médecin et du cas manuscrit ponctuel avocat, déployé chez le client (full Node.js).
  - ❌ **Document long en un seul prompt** : catastrophe après ~4 pages (contrat 7 pages — paragraphes manquants, sortie tronquée ; hallucination dès la 4e page constatée aussi sur le pipeline pymupdf4llm). Le long format passe par PyMuPDF ciblé + human-in-the-loop.
- **Rappel futur** : pour une note manuscrite ou un document court, la réponse est ollama/qwen vision local, pas un outil Python classique.

## Paramètres d'appel (production, API LangChain)

- temperature: 0, top_p: 1
- `enable_thinking: false` (chat_template_kwargs) si le modèle think est actif
- Gestion timeout / retry : à définir (squelette API : [../../use-cases/anonymisation/langchain-api.md](../../use-cases/anonymisation/langchain-api.md))

## Modèles installés (`ollama list`, octobre 2026)

- **frob/qwen3.5-instruct:9b** — modèle vision de référence (section ci-dessus). A remplacé `qwen3.5:9b` : la variante `instruct` est **instruction-tuned**, elle répond directement à l'instruction **sans bloq de raisonnement visible** (pas de `<think>` dans la sortie — pas de réflexion masquée, une réponse directe).
- **nomic-embed-text:latest** — modèle d'embedding, **ne sert à rien** (pas de RAG) mais très léger → gardé.

## Choisir le bon modèle : connaître sa machine EN PRIORITÉ

Avant tout choix de modèle local, connaître la **puissance de la machine** qui va le faire tourner (le modèle dépend du hardware, pas l'inverse).

**IMPORTANT — indiquer la langue cible dans le choix du modèle** : la langue influe à la fois sur la **taille** et la **performance** du modèle. Un modèle ~9B peut être excellent en anglais et nettement plus faible en français (jeu d'entraînement déséquilibré) : pour une autre langue que l'anglais, viser une taille supérieure ou vérifier que le modèle est explicitement multilingue. C'est un critère de choix au même rang que le hardware.

**Technique de Renaud (la simple)** : coller sa config machine dans un prompt et demander à l'IA de recommander le modèle adapté — pas besoin d'outil dédié.

- Outils si besoin : **llmfit** (installé sur le Mac de Renaud mais jugé trop complexe pour l'usage courant) et **canirun.ai** (https://www.canirun.ai).
- Commande : `llmfit system` → retourne la config de la machine (source de la config ci-dessous).

Config du Mac (sortie réelle `llmfit system`, octobre 2026) :

```
=== System Specifications ===
CPU: Apple M4 (10 cores)
Total RAM: 16.00 GB
Available RAM: 11.55 GB
RAM Bandwidth: ~211 GB/s (measured)
Backend: Metal
GPU: Apple M4 (unified memory, 11.84 GB GPU-available of 16.00 GB shared, Metal)
```

Lecture : Mac M4 16 Go, mémoire unifiée — ~11,84 Go adressables par le GPU (Metal). C'est ce plafond qui décide de la taille de modèle raisonnable (~9B type qwen3.5 tourne bien ; les très gros modèles sont hors de portée).

**Que veut dire « 9b »** : 9 milliards de paramètres — c'est la **taille limite pour le Mac M4 16 Go de RAM GPU**. Au-delà, le modèle ne tient pas dans la mémoire unifiée adressable par le GPU (~11,84 Go : le modèle chargé + son contexte doivent y tenir).

**Le nombre de cœurs** (M4 = 10 cœurs) compte surtout pour la vitesse de génération (tokens/s) et le pré-remplissage des prompts, mais ce n'est **pas** lui qui bloque : le vrai mur est la mémoire. Un modèle qui tient en RAM GPU tournera, plus ou moins vite selon les cœurs ; un modèle qui ne tient pas ne tourne pas du tout.

## Autres modèles locaux (mentionnés, non testés à fond)

Runtime Ollama via llama.cpp : `llama3.1`, `mistral`, `qwen2.5`… (inventaire libraries.md). Un modèle non testé n'entre pas dans le détail de cette fiche.

## Règles

- Une seule version vivante par besoin ; documenter ici chaque modèle réellement installé + testé + le besoin qu'il couvre.
- Le critère de décision est la **longueur/densité** du document, pas « IA locale oui/non » : court = validé, long = disqualifié.

## État

**vivant** — frob/qwen3.5-instruct:9b validé ×2 (court format) ; échec documenté long format ; paramètres production fixés (temperature 0).
