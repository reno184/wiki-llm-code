# Idée business : extraction de bilans/notes médicales par IA locale (vision)

Usage: piste business à ressortir si un prospect santé/médical se présente. Idée née d'un test OCR réel (octobre 2026), cadrée octobre 2026.

## L'idée

Un **médecin scanne un bilan ou une note de consultation dactylographiée** (1 page, 2 pages maximum, pas plus). Une **IA locale vision** (Ollama + frob/qwen3.5-instruct:9b, installée chez le médecin) transforme le scan en **Markdown**, puis en extrait les informations structurées : patient, motif, diagnostic, traitement, suivi → alimente le dossier médical, un CRM ou une base de données.

**Cible use case stricte** : extraction Markdown d'un scan de 1 à 2 pages dactylographiées. Ni les longs comptes rendus, ni les documents denses multi-pages (c'est le verrou qui a tué la piste avocat côté LLM, voir [ia-legaltech-contrat.md](ia-legaltech-contrat.md)).

## Pourquoi c'est pertinent

- **Confidentialité** : données de santé = le point bloquant du cloud. Le LLM tourne **chez le médecin** — rien ne sort de la machine, l'éditeur n'a même pas accès aux données. Argument de vente n°1 face aux API propriétaires.
- **Faisabilité prouvée ×2** : frob/qwen3.5-instruct:9b validé sur une fiche de lecture Harry Potter (PNG, contournement découpage sur zone dense) et sur une **fausse note médicale → parfait, réussite absolue** (voir [use-cases/ocr-llm/README.md](../use-cases/ocr-llm/README.md)). Le format cible (1-2 pages dactylographiées) est précisément la zone de validité du modèle.
- **Valeur** : le médecin ne veut pas du texte brut, il veut les champs structurés — extraction guidée par prompt, pas du simple OCR.
- **Le GPU est chez le client** : la contrainte « machines sans GPU » qui pèse sur le labo ne s'applique pas — on spécifie le prérequis matériel du cabinet (GPU correct ou Apple Silicon) dans l'offre.

## Architecture (full Node.js, zéro Python)

Le workflow production est **100% Node.js/JavaScript** :

- **API Express** chez le médecin, **LangChain** pour donner les instructions au modèle.
- **Ollama installé en local chez chaque médecin** pour l'accès GPU.
- **Modèle frob/qwen3.5-instruct:9b** pour la vision.
- Accès à l'Ollama server depuis un conteneur Docker via localhost (piège Docker : `localhost` conteneur ≠ hôte → `host.docker.internal`, à valider).
- Chaîne : scan → image (base64) → qwen vision → Markdown → extraction structurée par prompt.
- Squelette technique : [use-cases/anonymisation/langchain-api.md](../use-cases/anonymisation/langchain-api.md) (API Node + LangChain + qwen3.5, prompt chargé depuis fichier).

Aucun outil Python en prod (pas de PyMuPDF, pas d'OCRmyPDF, pas de Tesseract autonome — archivé, voir [libraries.md](../libraries.md)).

## Human-in-the-loop

Relecture humaine obligatoire : en médical, l'erreur coûte cher. Le LLM produit un brouillon Markdown + champs structurés, le médecin valide/corrige. Jamais de pipeline 100% automatique silencieux.

## Points à valider avant d'aller plus loin

- Volume et fréquence réels (combien de bilans/jour, quelle qualité de scan).
- Prérequis matériel du cabinet (GPU/Apple Silicon) — cadrage dans l'offre.
- Déploiement : Ollama + API chez le client (on-premise cabinet), modalités d'installation/mise à jour du modèle.
- Cible alternative : cabinets dentaires, kinés, ordonnances — même format court.

## Ne pas faire

- Ne pas envoyer des données de santé vers une API cloud tierce — RGPD, l'idée meurt si le local n'est pas respecté.
- Ne pas étendre aux documents longs/denses multi-pages : hors zone de validité du modèle (leçon contrat 7 pages).

## État

**vivant (idée non exploitée)** — cible recadrée (bilans/notes 1-2 pages dactylographiées), stack full Node.js définie (Express + LangChain + Ollama/qwen chez le client), faisabilité vision validée ×2, pas de prospect identifié.
