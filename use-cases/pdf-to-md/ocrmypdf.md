---
use-case: pdf-to-md
layer: backend-python
version: 1
---

# OCRmyPDF : prétraitement PDF

## Contexte d'usage
Toujours exécuté avant PyMuPDF4LLM. 2 rôles selon le PDF :

### Rôle 1 : layer texte invisible (PDF scannés)
Ajoute une couche texte invisible sous l'image → texte sélectionnable/extractible.

### Rôle 2 : optimisation du PDF
Nettoyage/optimisation avant conversion (défauts de scan, rotation, qualité).

## Les 2 types de PDF en entrée

1. **PDF natif** : texte déjà sélectionnable (généré par un logiciel) → l'OCR est inutile, voire dégrade le texte.
2. **PDF scanné** : simple image, aucun texte → l'OCR est **indispensable** pour créer le layer texte invisible.

## Options clés : gestion du texte existant

| Option | Comportement | Quand l'utiliser |
|---|---|---|
| *(défaut)* | Détecte seul : page avec texte → refuse/skip ; page image → OCR | Comportement de base, échoue en erreur si texte déjà présent |
| `--skip-text` | Saute les pages déjà textuelles, OCRise uniquement les images | **Mélange scannés + natifs** : jamais de dégradation |
| `--redo-ocr` | Ré-OCRise uniquement les pages avec un layer OCR existant de mauvaise qualité, ignore le texte natif | Layer OCR existant mais pourri |
| `--force-ocr` | Rastérise puis OCRise **toutes** les pages, même natives | **Dangereux** : détruit le texte natif (le remplace par l'interprétation OCR). Réservé au texte corrompu |

**Ordre de préférence** : `--skip-text` > `--redo-ocr` > `--force-ocr` (jamais force-ocr par défaut).

## Commande recommandée (chaîne pdf-to-md)

```bash
ocrmypdf -l fra --redo-ocr --optimize 1 input.pdf output.pdf
```

- `-l fra` : OCR en français (qualité de reconnaissance des accents)
- `--redo-ocr` : répare les mauvais layers OCR sans toucher au texte natif
- `--optimize 1` : optimisation légère et sûre (2/3 plus agressifs, risqués sur certains PDF)

**Variante si documents 100 % scannés** : `ocrmypdf -l fra --optimize 1 in.pdf out.pdf` (pas besoin de redo-ocr).
**Variante si mélange inconnu** : `ocrmypdf -l fra --skip-text --optimize 1 in.pdf out.pdf`.

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

Mêmes options que la CLI (`language`, `force_ocr`, `skip_text`, `redo_ocr`, `optimize`...). `force_ocr` = même danger que `--force-ocr` (rastérise tout, détruit le texte natif) — voir le tableau ci-dessus.

## Passage des options

CLI : `ocrmypdf [options] input.pdf output.pdf`. En Python, bibliothèque `ocrmypdf.api` ou `subprocess` avec les mêmes flags. Exécution : `pip install ocrmypdf` + moteur Tesseract installé au préalable.

## Notes
- Vérifier la sortie : OCRmyPDF logue les pages skipped/OCRisées — contrôle rapide avant PyMuPDF4LLM.
- Si PyMuPDF4LLM sort un Markdown vide/pourri → vérifier d'abord le layer texte (option mal choisie ou OCR raté).
