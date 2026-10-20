# OCRmyPDF : prétraitement PDF, ajout layer pour selectionner le text

> A tester sur wndows car mac le fait antivement


### Rôle 1 : layer texte invisible (PDF scannés)
Ajoute une couche texte invisible sous l'image → texte sélectionnable/extractible.

### Rôle 2 : optimisation du PDF
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


