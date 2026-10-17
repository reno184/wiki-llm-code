---
use-case: pdf-to-md
layer: backend-python
version: 0 (squelette)
---

# OCRmyPDF : prétraitement PDF

## Contexte d'usage
Toujours exécuté avant PyMuPDF4LLM. 2 rôles selon le PDF :

### Rôle 1 : layer texte invisible (PDF scannés)
Ajoute une couche texte invisible sous l'image → texte sélectionnable/extractible.

```bash
ocrmypdf input.pdf output.pdf
```

### Rôle 2 : optimisation du PDF
Nettoyage/optimisation avant conversion (défauts de scan, rotation, qualité).

## Commandes (à compléter avec les options réelles utilisées)

- `--force-ocr` : force l'OCR même si un layer texte existe (à n'utiliser que si l'OCR existant est mauvais)
- langue : `-l fra` pour documents français
- `--optimize 1|2|3` : optimisation de sortie

## Notes
- Point à documenter : quels flags exacts utilisés systématiquement, et comment détecter un PDF sans layer texte (`--skip-text` vs `--force-ocr`).
- À compléter avec le code/appel réel (CLI ou wrapper Python).
