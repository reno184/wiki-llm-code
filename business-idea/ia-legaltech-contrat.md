# Idée business : IA-legalTech — exploitation de contrats par IA locale

Usage: piste business à ressortir si un prospect juridique/DRH/notaire se présente. Idée née d'un test réel sur un contrat de travail (octobre 2026).

## L'idée

Un cabinet d'avocats, une DRH ou un notaire traite des **contrats** (travail, bail, commercial). Une **IA locale** (Ollama + modèle type qwen, rien dans le cloud) transforme le PDF en Markdown fidèle puis en extrait les informations structurées : parties, clauses, dates, obligations, résiliation → alimente un outil de gestion documentaire.

## Pourquoi c'est pertinent

- **Fidélité d'extraction prouvée** : test réel sur un contrat de travail de 7 pages — PyMuPDF4LLM sort un Markdown **100% fidèle** ("prédictif", pas un mot manquant, toutes les infos présentes). Voir [use-cases/pdf-to-md/README.md](../use-cases/pdf-to-md/README.md).
- **Confidentialité** : contrats = données sensibles (salaires, clauses). IA locale = argument RGPD face aux API cloud, comme pour la piste médicale.
- **Chaîne déjà prototypée** : OCRmyPDF → PyMuPDF4LLM → reformatage → extraction structurée par prompt.

## Verrou identifié (test réel, à lever avant d'aller plus loin)

Le reformatage sémantique du Markdown complet par un petit modèle local (qwen3.5:9b) **échoue au-delà de ~4 pages** : paragraphes perdus, sortie tronquée. → Obligation de **chunking** (découpage par sections) ou modèle plus gros. Voir [use-cases/reformat-markdown/README.md](../use-cases/reformat-markdown/README.md). Tant que ce verrou n'est pas levé, pas de promesse produit.

## Points à valider

- Chunking efficace (test à refaire avec découpage par sections).
- Précision exigée : en juridique, une clause perdue = risque → relecture humaine obligatoire, jamais 100% automatique.
- Cibles : avocats social, DRH (gestion des contrats), notaires, agences immobilières (baux).

## Ne pas faire

- Ne pas envoyer des contrats vers une API cloud tierce — RGPD, l'idée meurt si le local n'est pas respecté.

## État

**vivant (idée non exploitée)** — extraction Markdown validée (contrat 7 pages, 100% fidèle), reformatage long = verrou connu (leçon chunking), pas de prospect identifié.
