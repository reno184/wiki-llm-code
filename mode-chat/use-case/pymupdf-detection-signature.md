# Détection de signature dans un PDF — fingerprint et aiguillage

**Problème** : une signature ajoutée à un PDF n'est pas détectable par une seule méthode. Selon la plateforme (Aperçu macOS, DocuSign, Adobe Sign, scan...), elle vit à un endroit différent du fichier. Le fingerprint du buffer brut indique **où chercher en priorité**.

## Fingerprint du buffer (avant même d'analyser les pages)

Trois signaux structurels, par ordre d'utilité :

| Signal | Comment le lire | Aiguillage |
|---|---|---|
| **incremental_updates** | nb d'occurrences `startxref` - 1 dans les bytes | `> 0` → des objets ont été AJOUTÉS à la fin : chercher `annots()`, widgets, `/Sig` (cas DocuSign/Adobe Sign). `== 0` → fichier écrit en une passe : la signature (si présente) est **dans le flux de contenu** de la page → `get_drawings()`, `get_image_info()` |
| **/Sig** (signatures numériques) | scan des xrefs pour `FT == "/Sig"` + lire `SubFilter` | présent → plateforme pro, champs de signature. absent → signature manuscrite/scan/flattened |
| **producer** (métadonnées) | `doc.metadata["producer"]` | `Quartz PDFContext` = imprimé/flattened (Aperçu réécrit tout le fichier → 0 update malgré signature !). `iText`/`wkhtmltopdf` = généré natif. Scanner = signature dans une image |

**Piège majeur** : `incremental_updates == 0` ne veut PAS dire "pas de signature". Aperçu macOS **réécrit tout le fichier** au lieu d'ajouter une passe : la signature est "cuisinée" directement dans le contenu de la page. Cas réel : producer `Mac OS X Quartz PDFContext` + 0 update + signature présente sous forme de dessins `type="f"` noirs denses (122 items) dans `get_drawings()`.

## Capteurs de zones, dans l'ordre

```python
import pymupdf, re

# Fingerprint
n_update = max(0, len(re.findall(rb"startxref", pdf_bytes)) - 1)
# → aiguille les capteurs ci-dessous

# 1. Annotations : Ink = signature manuscrite, FreeText, Stamp, Widget
for a in page.annots(): ...          # a.rect, a.type

# 2. Images affichées y compris INLINE (xref == 0) :
for img in page.get_image_info(hashes=True, xrefs=True): ...  # img["bbox"]

# 3. Dessins vectoriels type signature Aperçu : fill noir, dense
for d in page.get_drawings():
    if d["type"] in ("f", "fs") and d["fill"] is not None and len(d["items"]) >= 10:
        ...                          # d["rect"] = zone signature
```

`get_images()` seul ne voit PAS les images inline → toujours `get_image_info(xrefs=True)`.

## Familles typiques

| Famille | Signaux | Où est la signature |
|---|---|---|
| Aperçu macOS (Annoter > Signature) | 0-1 update, pas de /Sig, parfois `Preview`/`Quartz` dans le buffer | Dessins `type="f"` denses noirs (vectorisée) ou image inline `xref=0` |
| Plateforme pro (DocuSign, Adobe Sign...) | updates > 0, `/Sig` présent, nom dans le buffer | Annotations standard (Stamp, Widget), update incrémental |
| Scan de papier signé | 0 update, producer scanner | Grosse image en bas de page (cas le plus simple) |

**Règle** : le fingerprint priorise les capteurs et ajuste les seuils, il ne court-circuite jamais la détection — toujours faire tourner les 3 capteurs.

## Liens croisés

- [pymupdf-images-position.md](pymupdf-images-position.md) — position des images/dessins (`get_drawings` → rect, bbox via `get_image_info`)
- [anonymous-rectangle-clip.md](anonymous-rectangle-clip.md) — suppression réelle par redaction
- [pymupdf_tech_anonymisation.md](pymupdf_tech_anonymisation.md) — anonymisation texte
