# Idée business : IA-legalTech — contrats, fiches de paie, M&A (avocats & cabinets)

Usage: piste business à ressortir si un prospect juridique/DRH/notaire/M&A se présente. Idée née d'un test réel sur un contrat de travail (octobre 2026), architecture recadrée octobre 2026.

## L'idée

Des cabinets d'avocats (dont M&A, merge & acquisition), DRH et notaires exploitent des **contrats et fiches de paie**. Le système transforme ces documents en texte exploitable avec **PyMuPDF en extraction ciblée** : l'avocat **sélectionne les zones dans une interface Vue.js**, PyMuPDF transforme en texte (clip + search_for + find_tables), **human-in-the-loop systématique** — pas de process 100% automatique visé.

Deux régimes documentaires dans le même produit :

1. **Contrats / fiches de paie (use case principal)** : PyMuPDF ciblé + interface Vue.js + relecture humaine. Voir [infra/pymupdf.md](../infra/pymupdf.md).
2. **Note manuscrite prise en réunion (ponctuel)** : le même pipeline vision que la piste médecin — scan → image → qwen vision → Markdown (Express + LangChain + Ollama, installés chez le client).

## Pourquoi cette architecture

- **La fidélité automatique sur le long format est un échec constaté** : reformatage sémantique d'un contrat 7 pages par qwen3.5:9b → catastrophe après ~4 pages. Le 100% automatique local n'est pas atteignable avec les modèles actuels sur les machines cibles (peu de GPU) → assumer le human-in-the-loop comme **choix d'architecture**, pas un pis-aller.
- **Tesseract (autonome) et Docling écartés** : résultats décevants, archivés (trace [_archive/tesseract-docling-standby.md](../_archive/tesseract-docling-standby.md)).
- **Confidentialité** : contrats = données sensibles. PyMuPDF = déterministe, pas de LLM dans la boucle sur le cas principal ; le LLM vision (cas manuscrit ponctuel) tourne en local chez le client, rien dans le cloud.
- **Fidélité d'extraction prouvée** : PyMuPDF4LLM sort un Markdown 100% fidèle d'un contrat 7 pages — la matière première est fiable, c'est la transformation automatique complète qui ne l'est pas (voir [use-cases/pdf-to-md/README.md](../use-cases/pdf-to-md/README.md)).

## Stack

- **Cas principal** : PyMuPDF (labo Python, extraction ciblée — clip, search_for, find_tables pour les grilles de paie, get_drawings pour les tableaux sans bordures) + **interface Vue.js** de ciblage des zones + validation humaine.
- **Cas manuscrit ponctuel** : full Node.js — Express + LangChain + Ollama/frob-qwen3.5-instruct:9b chez le client, même pattern que la piste médecin (voir [business-idea/ia-medical-ocr-consultation.md](ia-medical-ocr-consultation.md)).
- Point ouvert : le traitement contrat doit-il tourner chez l'avocat (→ micro-service Python Docker) ou en back-office ? À trancher au premier prospect.

## Points à valider

- Ergonomie du ciblage Vue.js (sélection de zones, prévisualisation du texte extrait).
- Précision exigée : en juridique, une clause perdue = risque → relecture humaine obligatoire, inscrite dans l'architecture.
- Cibles : avocats social, M&A (due diligence = lots de contrats à exploiter), DRH, notaires, agences immobilières (baux).

## Ne pas faire

- Ne pas envoyer des contrats vers une API cloud tierce — secret professionnel, l'idée meurt si le local n'est pas respecté.
- Ne pas promettre du 100% automatique sur les documents longs.

## État

**vivant (idée non exploitée)** — architecture recadrée : PyMuPDF ciblage Vue.js + human-in-the-loop pour le cas principal, pipeline vision mutualisé avec la piste médecin pour le manuscrit ponctuel. Pas de prospect identifié.

## Killer features « avocats » (retours terrain cabinets, oct. 2026)

Le besoin n'est pas le juridique pur mais le **travail de masse sur les pièces** (PDF, scans). Classement par valeur :

1. **Anonymisation / pseudonymisation intelligente (top 1, RGPD)** — anonymisation **contextuelle et sélective** : masquer le client A et ses entreprises mais garder la partie adverse ; fichier de correspondance généré (M. X → « Monsieur [A] ») pour rester lisible dans la rédaction. Obligatoire en France avant publication/partage des décisions de justice.
2. **Document splitting automatique** — un méga-PDF de 300 pages (pièces scannées à la chaîne) découpé en documents individuels avec **nommage intelligent** (`2024-01-15_Contrat_de_travail_ClientX.pdf`) : détection IA du début de chaque pièce.
3. **Extraction de données → Excel** — matrices de comparaison en un clic : 36 fiches de paie → *période / brut / heures supp / cumul imposable* ; 100 factures → *n° / date / HT / TTC / échéance*. **Killer : export Excel avec hyperliens vers le bloc source du PDF** (traçabilité pièce).
4. **Détection de signatures & métadonnées par lot** (due diligence, gros contentieux) — filtre rapide : signé / non signé (manuscrit ou tampon), identité des signataires et dates, alerte contrats à échéance sans tacite reconduction.
5. **Recherche transversale de clauses** — 50 contrats → « extrais toutes les clauses de limitation de responsabilité ou d'exclusivité » dans un document comparatif.
6. **Chronologie automatique des faits** — toutes les dates extraites de toutes les pièces (mails, contrats, RAR) → **fiche chronologique synthétique** pour la plaidoirie.

Points communs : **traitement par lots de documents scannés** (OCR requis, pas seulement texte natif), **sorties structurées traçables** (lien retour à la source), confidentialité locale — cohérent avec l'architecture de cette fiche. La feature 1 (anonymisation) croise [pymupdf_tech_anonymisation.md](../use-case/pymupdf_tech_anonymisation.md).
