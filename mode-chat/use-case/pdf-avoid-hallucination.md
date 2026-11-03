# Éviter les hallucinations quand un PDF part dans un LLM

**Problème** : avant d'envoyer un contrat à un LLM, faut-il supprimer les zones graphiques (signature, tampon, logo, image) pour réduire les tokens et le risque d'hallucination ? Réponse courte : **le gain de tokens est quasi nul, le gain anti-hallucination et anti-PII est réel**.

## Tokens : dépend du mode d'ingestion, pas de la suppression

| Mode d'ingestion | Les zones supprimées réduisent-elles les tokens ? |
|---|---|
| PDF → texte (`get_text()` → prompt) | ❌ Quasi zéro : images et drawings n'ont **jamais** produit de tokens. Seul le texte des zones encadrées compterait |
| PDF natif à une API (service convertit en images + texte) | ⚠️ Partiel : la zone devient blanche mais la page garde sa taille → coût visuel inchangé |
| Pipeline vision/OCR (pages rendues en PNG) | ⚠️ Dimensions inchangées = mêmes tokens d'image. L'OCR ne lit plus la signature — gain de **bruit**, pas de tokens |

**La vraie réduction de tokens** se cache dans les **headers/footers répétés** : du texte facturé à chaque page d'un contrat de 20 pages. Pour ça, `apply_redactions(text=1)` ciblé sur ces zones (ou filtrage à l'extraction), pas `text=0` global.

## Pourquoi la suppression réduit quand même le risque d'hallucination

Quel que soit le mode d'ingestion :

- Une **signature manuscrite contient souvent le nom du signataire** → un modèle vision "lit" un nom, ou en invente un, et l'injecte dans sa réponse (fuite PII)
- **Tampons, logos, cases cochées** = bruit graphique qui pousse à halluciner du contexte ("cacheté par X", "société au logo bleu")
- Moins de contenu hors-sujet = attention concentrée sur le contrat utile

Conclusion : la suppression de zones est **d'abord un geste de sécurité anti-PII et anti-hallucination**. L'allègement du fichier est un bonus.

## Règles pratiques

```python
# Suppression réelle dans le fichier, pas seulement visuelle :
doc.save("sortie.pdf", garbage=3, deflate=True)  # garbage=3 purge les objets orphelins

# Choisir text selon la zone :
# - zones graphiques (signature, logo) : text=0 (le texte de la zone reste)
# - header/footer : text=1 (c'est du texte répété, coût réel en tokens)
```

- Gain de poids : significatif si la signature était une **image** ; quasi nul si elle est **vectorisée** (quelques centaines d'opérateurs de dessin)
- Toujours vérifier après save que le contenu retiré n'est plus dans le binaire

## Liens croisés

- [pymupdf-detection-signature.md](pymupdf-detection-signature.md) — détecter les zones signature avant de les supprimer (fingerprint → capteurs)
- [anonymous-rectangle-clip.md](anonymous-rectangle-clip.md) — pourquoi la redaction supprime réellement (add_redact_annot + apply_redactions)
- [pymupdf_tech_anonymisation.md](pymupdf_tech_anonymisation.md) — anonymisation du texte restant
- [workflow-pdf2md.md](workflow-pdf2md.md) — pipeline complet PDF → texte pour LLM
