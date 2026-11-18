# OCR Gateway — router les PDF scannés vers OCRmyPDF

**Problème** : à l'import de factures/contrats, distinguer les PDF scannés (1 image pleine-page, pas de texte natif) des PDF natifs, pour ne lancer l'OCR que quand c'est nécessaire. Probe de quelques ms/page.

## Probe : 2 conditions combinées, pas 1 seule

```python
import pymupdf

def est_scan(page) -> bool:
    """Un scan = image(s) couvrant la page + quasi aucun texte natif."""
    # 1. Image(s) affichée(s) couvrant ~toute la page
    area_page = abs(page.rect)
    couverture = 0.0
    for img in page.get_image_info():
        couverture += abs(pymupdf.Rect(img["bbox"])) / area_page
    if couverture < 0.8:          # < 80% de la page → pas un scan
        return False

    # 2. Presque pas de texte natif
    if len(page.get_text().strip()) > 20:
        return False              # texte natif → PDF natif

    return True
```

**Pourquoi la 2e condition est indispensable** : des PDF métier (factures EDI, plateformes de facturation) sont des PDF natifs AVEC une image de fond pleine-page (papier à en-tête scanné) + du vrai texte par-dessus. Les passer dans OCRmyPDF serait une régression (l'OCR est moins bon que le texte natif).

`get_image_info()` et pas `get_images()` : il faut les images **affichées** avec leur bbox, y compris inline.

## Le switcher

```
Import PDF → probe est_scan() par PAGE
      │
      ├── SCAN ──► OCRmyPDF ──► PDF texte ──► pipeline PyMuPDF
      │
      └── NATIF ─► pipeline PyMuPDF direct (get_text, détection zones...)
```

```python
import subprocess

def ocr_scan(pdf_path: str, out_path: str) -> None:
    subprocess.run([
        "ocrmypdf",
        "--deskew",           # redresse les scans de travers
        "--rotate",           # réoriente les pages à l'envers
        "--skip-text",        # pages déjà textuelles → ne pas retoucher (PDF mixtes)
        "--output-type", "pdf",  # pas pdfa si retravail PyMuPDF ensuite
        pdf_path, out_path,
    ], check=True)
```

## Intégration : le résultat du probe vit avec le fingerprint d'import

Le probe est calculé **une fois à l'import** et stocké dans les métadonnées (MinIO) aux côtés du fingerprint de détection de signature :

```json
{
  "isSig": false,
  "incremental_updates": 0,
  "producer": "Ricoh Aficio MP C3000",
  "is_scan": true
}
```

Le gateway lit ensuite la métadonnée — le routing n'est pas recalculé à chaque traitement. `producer` confirme souvent le diagnostic (nom du scanner).

## Pièges

- **Tester par PAGE, pas par document** : un PDF de 5 pages dont 4 natives et 1 annexe scannée est un cas réel. `--skip-text` gère bien ce cas page par page
- **Après OCR, la détection de zones change** : la signature manuscrite scannée est *dans* l'image pleine-page — annots/drawings ne la voient pas. Sur un scan, la zone signature = rect bas de page (à traiter comme cas à part dans la gateway)
- OCRmyPDF coûte quelques secondes/page : ne le lancer que si `is_scan` est vrai

## Liens croisés

- [pymupdf-detection-signature.md](pymupdf-detection-signature.md) — fingerprint d'import (isSig, incremental_updates, producer), même métadonnée
- [ocr-on-png.md](ocr-on-png.md) — OCR sur images directes
- [pdf-text-selectionable.md](pdf-text-selectionable.md) — détecter si un PDF a du texte natif
- [workflow-pdf2md.md](workflow-pdf2md.md) — pipeline complet PDF → texte pour LLM
