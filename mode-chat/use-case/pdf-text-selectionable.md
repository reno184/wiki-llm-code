# OCRmyPDF : prétraitement PDF, ajout layer pour selectionner le text

> A tester sur windows car macOS le fait nativement

Ajoute une couche texte invisible sous l'image → texte sélectionnable/extractible.
Nettoyage/optimisation avant conversion (défauts de scan, rotation, qualité).

## Les 2 types de PDF en entrée

1. **PDF natif** : texte déjà sélectionnable (généré par un logiciel) → l'OCR est inutile, voire dégrade le texte.

2. **PDF scanné** : simple image, aucun texte → l'OCR est **indispensable** pour créer le layer texte invisible.

## Appel Python (API ocrmypdf)

```python
from pathlib import Path
import ocrmypdf

current_dir = Path(__file__).parent
input_pdf = current_dir / "document-source.pdf"
output_pdf = current_dir / "document-dest.pdf"

print("Lancement de l'OCR...")

ocrmypdf.ocr(
    input_file=input_pdf,
    output_file=output_pdf,
    language="fra",
    force_ocr=True,
    progress_bar=True
)

print("OCR terminé")
```

## Contenu d'origine (leçons conservées à titre de trace)

Gonflement de taille après OCR :

- `force_ocr` re-rasteurise TOUTES les pages (même celles qui ont déjà du texte) → explosion de la taille.
- PDF/A : profils exigés + polices embarquées → alourdit.
- Leviers de réduction, dans l'ordre : **skip-text** (n'OCRise que les pages sans texte) > **redo_ocr** (refait seulement les zones déjà OCRisées) > force_ocr (jamais par défaut).
- Compression : `jbig2enc` (images N&B), `pngquant` (couleur), `--optimize`.


# Tesseract (pymupdf) vs ocrmypdf

- **Tesseract en direct** : OCR bas niveau, déjà en retrait derrière ocrmypdf (qui l'utilise en interne et apporte le prétraitement). L'utiliser nu n'apporte rien de plus que la chaîne OCRmyPDF existante pour les scannés.

## todo

- faire dans le projet proto, un essai, charger des pdf selctionnable et non selectionnables, api python renvoi true ou false si selectionnable ou pas.

## État

**vivant** 


## tags
[python,OCRmyPDF,OCR, convert-pdf]