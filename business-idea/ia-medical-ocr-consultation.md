# Idée business : extraction d'infos médicales par IA locale (OCR)

Usage: piste business à ressortir si un prospect santé/médical se présente. Idée née d'un test OCR réel (octobre 2026).

## L'idée

Un **médecin scanne un compte rendu de consultation** (papier ou PDF). Une **IA locale (Ollama / modèle vision type qwen)** extrait les informations structurées : patient, motif, diagnostic, traitement, suivi → alimente un dossier médical, un CRM ou une base de données.

## Pourquoi c'est pertinent

- **Confidentialité** : données de santé = le point bloquant du cloud. Une IA locale (rien ne sort de la machine) lève l'objection RGPD/HDS d'un coup — c'est l'argument de vente n°1 face aux API propriétaires.
- **Faisabilité prouvée en local** : OCR d'un PNG avec Ollama + frob/qwen3.5-instruct:9b testé et concluant, voir [use-cases/ocr-llm/README.md](../use-cases/ocr-llm/README.md).
- **Valeur** : le médecin ne veut pas du texte brut, il veut les champs structurés — c'est de l'extraction guidée par prompt, pas du simple OCR.

## Points à valider avant d'aller plus loin

- Volume et fréquence réels (combien de comptes rendus/jour, quelle qualité de scan).
- Précision exigée : en médical, l'erreur coûte cher → relecture humaine obligatoire, pipeline jamais 100% automatique.
- Hébergement : local cabinet (mac mini/serveur) vs on-premise hébergé.
- Cible alternative : mêmes besoins en cabinet dentaire, kiné, ordonnances.

## Ne pas faire

- Ne pas envoyer des données de santé vers une API cloud tierce - RGPD — l'idée meurt si le local n'est pas respecté.

## État

**vivant (idée non exploitée)** — test technique validé côté OCR, pas de prospect identifié.
