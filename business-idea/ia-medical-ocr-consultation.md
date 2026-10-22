# Idée business : extraction de bilans/notes médicales par IA locale (vision)

Usage: piste business à ressortir si un prospect santé/médical se présente. Idée née d'un test OCR réel (octobre 2026), cadrée octobre 2026.

## L'idée

Un **médecin scanne un bilan ou une note de consultation dactylographiée** (1 page, 2 pages maximum, pas plus). 
Une **IA locale vision** (Ollama + frob/qwen3.5-instruct:9b, installée chez le médecin) transforme le scan en **Markdown**, puis en extrait les informations structurées.

**Cible use case stricte** : extraction Markdown d'un scan de 1 à 2 pages dactylographiées.

## Pourquoi c'est pertinent

- **Confidentialité** : données de santé = le point bloquant du cloud. 
Le LLM tourne **chez le médecin** 

- **Faisabilité prouvée ×2** : frob/qwen3.5-instruct:9b validé sur une fiche de lecture Harry Potter (PNG, contournement découpage sur zone dense) et sur une **fausse note médicale → parfait, réussite absolue**.
- **Le GPU est chez le client** : la contrainte « machines sans GPU » qui pèse sur le labo ne s'applique pas — on spécifie le prérequis matériel du cabinet (GPU correct ou Apple Silicon) dans l'offre.

## Architecture (full Node.js, zéro Python)

Le workflow production est **100% Node.js/JavaScript** :

- **API Express** chez le médecin, **LangChain** pour donner les instructions au modèle.
- **Ollama installé en local chez chaque médecin** pour l'accès GPU.
- **Modèle frob/qwen3.5-instruct:9b** pour la vision.

## Ne pas faire

- Ne pas envoyer des données de santé vers une API cloud tierce — RGPD, l'idée meurt si le local n'est pas respecté.
- Ne pas étendre aux documents longs/denses multi-pages : hors zone de validité du modèle (leçon contrat 7 pages).

## État

**vivant**
