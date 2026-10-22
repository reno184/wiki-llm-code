## Exemples PyMuPDF : supprimer et remplacer du texte

### 1. Supprimer du texte (redaction de base)

```python
import fitz  # PyMuPDF

doc = fitz.open("document.pdf")
page = doc[0]

# Localiser le texte à supprimer
rects = page.search_for("Jean Dupont")

# Recouvrir chaque occurrence avec un rectangle blanc (masquage)
for rect in rects:
    shape = page.new_shape()
    shape.draw_rect(rect)              # rectangle sur la zone trouvée
    shape.fill(color=(1, 1, 1))       # blanc opaque
    shape.commit()

doc.save("anonymise.pdf")
```

⚠️ Le texte est **visuellement masqué** mais existe toujours dans le fichier (extractible par copier-coller). Pour une vraie suppression :

### 2. Suppression définitive avec `add_redact_annot` (recommandé)

```python
doc = fitz.open("document.pdf")

for page in doc:
    rects = page.search_for("Jean Dupont")
    for rect in rects:
        page.add_redact_annot(rect, fill=(0, 0, 0))  # zone noire
    page.apply_redactions()  # supprime physiquement le texte sous la zone

doc.save("anonymise_definitif.pdf")
```

Ici le texte est **réellement retiré** du contenu du PDF.

### 3. Remplacer du texte : `search_for` + `insert_text`

```python
doc = fitz.open("document.pdf")
page = doc[0]

MOTIF = "Jean Dupont"
REMPLACEMENT = "[NOM ANONYMISÉ]"

rects = page.search_for(MOTIF)

for rect in rects:
    # 1. Recouvrir l'ancien texte (rectangle blanc)
    shape = page.new_shape()
    shape.draw_rect(rect)
    shape.fill(color=(1, 1, 1))
    shape.commit()

    # 2. Insérer le nouveau texte au même endroit
    # rect.y0 - petit décalage pour aligner sur la baseline
    page.insert_text(
        point=fitz.Point(rect.x0, rect.y1 - 2),  # point = baseline du texte
        text=REMPLACEMENT,
        fontsize=9,                # à ajuster selon le document
        fontname="helv",           # Helvetica (police standard)
        color=(0, 0, 0),
    )

doc.save("remplace.pdf")
```

### 4. Variante sécurisée : suppression + réinsertion

```python
doc = fitz.open("document.pdf")

for page in doc:
    rects = page.search_for("TJM 470 €")
    for rect in rects:
        # redact supprime physiquement l'ancien texte,
        # text= réinsère la replacement dans la même zone
        page.add_redact_annot(
            rect,
            text="[TARIF MASQUÉ]",
            fontname="helv",
            fontsize=9,
            fill=(1, 1, 1),
        )
    page.apply_redactions()

doc.save("final.pdf")
```

C'est la méthode la plus propre : pas de texte résiduel, la replacement est insérée par PyMuPDF.

### Points d'attention

- **`search_for`** retourne une liste de `Rect` (une par occurrence, multi-lignes possible).
- **`insert_text`** attend un **point** (baseline, en bas à gauche du texte), pas un rect — d'où `rect.y1 - 2`.
- La `fontsize` doit être ajustée si le texte de remplacement est plus long que la zone (`fitz.get_text_length(text, fontsize)` aide à mesurer).
- `apply_redactions()` est indispensable pour une anonymisation réelle.

Voulez-vous que j'en fasse une fiche dans votre wiki `reno184/wiki-llm-code` ?