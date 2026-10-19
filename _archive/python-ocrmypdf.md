# OCRmyPDF (gonflement de taille) — ARCHIVÉ

**Archivé (oct. 2026)** : décision de rester **uniquement sur PyMuPDF** pour toute la chaîne PDF. Le périmètre produit exclut les scannés (décision pdf-to-text : l'avocat ne travaille pas sur des contrats scannés) → le besoin d'un layer OCR amont disparaît, et avec lui la raison d'être de cette fiche.

**Réveil si** : besoin réel de traiter des documents scannés (un client apporte des PDF image). PyMuPDF n'a pas de moteur OCR intégré — c'est Tesseract via OCRmyPDF (ou équivalent) qu'il faudra réintroduire. Le contenu ci-dessous reste alors pertinent.

---

## Contenu d'origine (leçons conservées à titre de trace)

Gonflement de taille après OCR :

- `force_ocr` re-rasteurise TOUTES les pages (même celles qui ont déjà du texte) → explosion de la taille.
- PDF/A : profils exigés + polices embarquées → alourdit.
- Leviers de réduction, dans l'ordre : **skip-text** (n'OCRise que les pages sans texte) > **redo_ocr** (refait seulement les zones déjà OCRisées) > force_ocr (jamais par défaut).
- Compression : `jbig2enc` (images N&B), `pngquant` (couleur), `--optimize`.

## État

**archivé (écarté)** — chaîne retenue = PyMuPDF seul sur PDF natifs ; aucune OCR dans le pipeline actuel.
