# PyMuPDF — extraction de texte, ciblage géométrique, tableaux et liens

Stack: Python + pymupdf (fitz), exécution labo Docker (voir infra/python-docker-labo.md).
Usage: boîte à outils d'extraction ciblée depuis un PDF natif (texte, styles, zones, tableaux, liens) — complément de use-cases/pdf-to-md/ (chaîne OCRmyPDF → PyMuPDF4LLM) quand la conversion intégrale ne suffit pas. Rappel hiérarchie (fiche use-cases/ocr-llm) : PDF natif → PyMuPDF, scan → OCRmyPDF, image complexe/manuscrit → Ollama/qwen.

## Les 3 modes de lecture du texte

### 1. `page.get_text("blocks", sort=True)`

Retourne une liste de tuples `(x0, y0, x1, y1, texte, n°_bloc, type_bloc)` — un bloc ≈ zone/paragraphe visuel.

- `sort=True` trie dans l'ordre visuel de lecture (haut → bas). Sans ça, l'ordre suit la structure interne du PDF → texte en désordre.
- Cas d'usage : paragraphes entiers dans l'ordre, repérage de sections par leur zone géométrique.
- Intermédiaire entre le texte brut et le dictionnaire complet.

### 2. `page.get_text("dic")`

Dict hiérarchique `page → blocks → lines → spans`. La source de tout : chaque niveau porte ses coordonnées, les **spans** portent le style (police, taille, gras, couleur).

- C'est ce qui permet de détecter un **titre** (grande taille / gras) vs un paragraphe → cœur d'une conversion PDF → Markdown fidèle.
- Contrepartie : verbeux, à parcourir par boucles imbriquées.

### 3. Boucles imbriquées block → line → span

```python
import pymupdf

doc = pymupdf.open("doc.pdf")

for page in doc:
    for block in page.get_text("dic")["blocks"]:
        if block["type"] != 0:
            continue
        for line in block["lines"]:
            for span in line["spans"]:
                print(span["text"], span["size"], span["flags"])
```

- **block** ≈ zone/paragraphe, **line** ≈ ligne visuelle (générée par PyMuPDF), **span** = segment homogène en style.
- Piège : filtrer les blocks images (`type != 0`) sinon KeyError sur `block["lines"]`.

## Ciblage géométrique

### 4. `clip` + `pymupdf.Rect(rx0, ry0, rx1, ry1)`

`Rect` = rectangle en coordonnées page, origine **en haut à gauche**, y vers le bas.

```python
cr = pymupdf.Rect(50, 100, 300, 200)
texte_zone = page.get_text(clip=cr)
```

Cas d'usage : extraire une colonne, un en-tête, une zone d'adresse sans lire toute la page.

### 5. `page.search_for("Répondre aux questions")`

Retourne une **liste de Rects** (un par occurrence). Le pont texte → géométrie : le Rect trouvé alimente `clip` (texte autour) ou se combine avec `find_tables()` (tableau qui suit la question). Combo pour documents à structure répétitive (questionnaires, formulaires).

## Tableaux et liens

### 6. `page.find_tables()`

Détecte les tableaux (bordures ou alignements).

- `page.find_tables().tables` → liste ; `table.extract()` → liste de listes (lignes × colonnes), prêt pour CSV/pandas.
- `table.bbox` pour les coordonnées, `table.header` pour l'en-tête.
- Piège : détection non infaillible (sans bordures, cellules fusionnées) → vérifier visuellement sur un échantillon.

### 7. `page.get_links()`

Liste les liens de la page : chaque lien porte `kind` (LINK_URI, LINK_GOTO…), `from` (Rect cliquable) et la cible (`uri` ou `page`). Indispensable pour ne pas perdre les liens lors d'une conversion Markdown.

## Mémo

Le `dic` donne structure et style, `search_for` + `Rect` + `clip` ciblent, `find_tables` les tableaux, `get_links` les liens. Ensemble = pipeline d'extraction ciblée PDF → Markdown, alternative à la conversion intégrale à l'aveugle.

## État

**vivant** — notions consolidées (2026-10), à valider avec les tests du proto-pdf-to-md.md.
