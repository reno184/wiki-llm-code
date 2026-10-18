# OCRmyPDF — pourquoi le fichier OCRisé gonfle (368 Ko → 2,4 Mo)

Stack: Python + ocrmypdf (language fra, force_ocr, sortie PDF/A), exécution labo Docker (voir infra/python-docker-labo.md).
Usage: comprendre l'augmentation de taille après OCR et les leviers pour la réduire. Complément de use-cases/pdf-to-md/ocrmypdf.md (choix des options skip-text/redo-ocr/force-ocr).

## Ce que fait l'OCR

L'OCR rend un document image ou un PDF numérisé interactif, recherchable et sélectionnable :

1. **Analyse de l'image** : extraction des pages en images haute résolution, repérage des zones de texte.
2. **Reconnaissance du texte** : identification des lettres/mots (ici en français via language="fra").
3. **Calque de texte invisible (Invisible Text Layer)** : couche de texte injectée par-dessus les images d'origine, calée sur l'emplacement des mots → copier/coller et Ctrl+F possibles.
4. **Normalisation PDF/A** : conversion au format d'archivage long terme → intégration de polices, métadonnées de profil, profils de couleur.

## Pourquoi 368 Ko → 2,4 Mo

1. **force_ocr=True (re-rasteurisation)** : ocrmypdf re-rend toutes les pages en nouvelles images rasterisées avant OCR, puis les ré-encode dans le PDF final. Si les images d'origine étaient compressées ou vectorielles/optimisées, les nouvelles images (non compressées ou sans perte) prennent beaucoup plus de place.
2. **Calque de texte + polices embarquées** : le texte reconnu et les polices nécessaires (même invisibles) sont intégrés (embedded) dans le PDF destination.
3. **Profils PDF/A** : la norme impose des métadonnées strictes et des profils ICC — plusieurs centaines de Ko à eux seuls.
4. **Pas de compression poussée** : logs `Recompressing JPEGs 0%` et JBIG2 non appliqué → les nouvelles images restent à leur taille d'export brute.

## Réduire la taille

- **PDF source avec images + OCR ciblé** : remplacer force_ocr=True par **redo_ocr=True**, ou laisser le défaut **skip_text=True** (OCR uniquement sur les pages sans texte).
- **Compression d'image** : activer l'optimisation, ou installer **jbig2enc** (images monochromes) / **pngquant** — ocrmypdf les utilise automatiquement.

## Ordre de préférence (rappel fiche pdf-to-md)

skip-text > redo-ocr > force-ocr. Jamais force_ocr par défaut : il détruit le texte natif ET gonfle le fichier.

## État

**vivant** — observation issue du labo OCR (2026-10), à consolider avec les tests du proto-pdf-to-md.md.
