# PyMuPDF — extraction de texte, ciblage géométrique, images, tableaux et liens

Stack: Python + pymupdf , exécution labo Docker (voir infra/python-docker-labo.md).
Usage: boîte à outils d'extraction ciblée depuis un PDF natif (texte, styles, zones, images, tableaux, liens) — complément de use-cases/pdf-to-md/ (chaîne OCRmyPDF → PyMuPDF4LLM) quand la conversion intégrale ne suffit pas. Rappel hiérarchie (fiche use-cases/ocr-llm) : PDF natif → PyMuPDF, scan → OCRmyPDF, image complexe/manuscrit → Ollama/qwen.


> ⚠️ **Faiblesse : licence AGPL** — attention si usage commercial non-libre. Pour commercialiser, il faut une licence commerciale Artifex (payante).

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

### 4. `page.get_text("words")`

Granularité maximale : liste de mots avec coordonnées `(x0, y0, x1, y1, mot, ...)`, `sort=True` disponible. Base des regroupements maison (regrouper par ligne via les y proches).

## Ciblage géométrique

### 5. `clip` + `pymupdf.Rect(rx0, ry0, rx1, ry1)`

`Rect` = rectangle en coordonnées page, origine **en haut à gauche**, y vers le bas.

```python
cr = pymupdf.Rect(50, 100, 300, 200)
texte_zone = page.get_text(clip=cr)
```

Cas d'usage : extraire une colonne, un en-tête, une zone d'adresse sans lire toute la page.

### 6. `page.search_for("Répondre aux questions")`

Retourne une **liste de Rects** (un par occurrence). Le pont texte → géométrie : le Rect trouvé alimente `clip` (texte autour), `get_pixmap(clip=…)` (image de la zone) ou se combine avec `find_tables()` (tableau qui suit la question). Combo pour documents à structure répétitive (questionnaires, formulaires).

## Images

### 7. Lister et extraire : `get_images` + `extract_image`

`page.get_images(full=True)` liste les images référencées (xref, dimensions, compression) mais **sans position**. `doc.extract_image(xref)` sort l'image brute (bytes + extension).

```python
for img in page.get_images(full=True):
    pix = doc.extract_image(img[0])
    open(f"img_{img[0]}.{pix['ext']}", "wb").write(pix["image"])
```

### 8. Positionner : `get_image_rects(xref)` / `get_image_info()`

Les **positions** des images sur la page — l'équivalent de `search_for` pour les images. Indispensable pour savoir quelle image correspond à quelle section de texte.

### 9. Rendu de page : `get_pixmap`

- `page.get_pixmap(dpi=150)` → page **entière** en image (PNG via `pix.save()` / `pix.tobytes()`) : utile pour l'OCR ou pour donner la page à un LLM vision (use-case ocr-llm, base64 → ollama).
- `page.get_pixmap(clip=cr, dpi=200)` → rendu **d'une zone seulement** : la leçon de la fiche ocr-llm (découper les zones denses avant envoi au modèle).

## Structure et mise en page

### 10. `page.get_drawings()` + `cluster_drawings()`

Tous les objets vectoriels (lignes, rectangles, courbes) avec couleurs/épaisseurs. Permet de détecter des **tableaux sans bordures de texte** (lignes de séparation), surlignages, cadres. `cluster_drawings()` regroupe les dessins proches (zones de tableau).

### 11. `doc.get_toc()`

La **table des matières** du PDF (bookmarks) → hiérarchie des titres gratuite, sans deviner via les tailles de police. À croiser avec les spans du `dic` pour fiabiliser la détection de titres.

## Tableaux et liens

### 12. `page.find_tables()`

Détecte les tableaux (bordures ou alignements).

- `page.find_tables().tables` → liste ; `table.extract()` → liste de listes (lignes × colonnes), prêt pour CSV/pandas.
- `table.bbox` pour les coordonnées, `table.header` pour l'en-tête.
- Piège : détection non infaillible (sans bordures, cellules fusionnées) → vérifier visuellement sur un échantillon. Alternative pour les tableaux sans bordures de texte : `get_drawings()` (point 10).

### 13. `page.get_links()`

Liste les liens de la page : chaque lien porte `kind` (LINK_URI, LINK_GOTO…), `from` (Rect cliquable) et la cible (`uri` ou `page`). Indispensable pour ne pas perdre les liens lors d'une conversion Markdown.

## Autres

- **`page.annots()`** — annotations (surlignage, notes) : un surlignage = Rect + couleur, souvent porteur de sens (ex. anonymisation : repérer ce qui a été marqué).
- **`doc.metadata`** — auteur, titre, dates.
- **`page.rotation`** — piège fréquent : les coordonnées Rect sont dans le système de la page, à tester sur une page tournée.

## Table de correspondance par besoin

| Besoin | Méthodes |
|---|---|
| Page entière → LLM vision (OCR complexe) | `get_pixmap(dpi=…)` → base64 → ollama |
| Zone dense découpée (leçon ocr-llm) | `search_for` ou `get_image_rects` → `Rect` → `get_pixmap(clip=…)` |
| Images à extraire d'un PDF | `get_images` + `extract_image` |
| Tableau sans bordures de texte | `get_drawings()` / `cluster_drawings()` |
| Titres fiables | `doc.get_toc()` croisé avec les spans du `dic` |
| Anonymisation (zones marquées) | `annots()` → `Rect` → `clip` |

## Mémo

Le `dic` donne structure et style, `search_for` + `Rect` + `clip` ciblent, `get_images`/`extract_image`/`get_pixmap` gèrent les images, `find_tables` les tableaux, `get_links` les liens. Ensemble = pipeline d'extraction ciblée PDF → Markdown, alternative à la conversion intégrale à l'aveugle (pymupdf4llm combine plusieurs de ces briques en interne).

## Anonymisation

PyMuPDF permet de manipuler directement les PDF — dessiner sur le document, insérer du texte, ou supprimer du texte existant. Grâce à la fonction search_for, on peut localiser automatiquement des zones sensibles et donc réaliser une anonymisation de PDF, sans passer par un outil externe.


## État

**vivant** — notions consolidées (2026-10), à valider avec les tests du proto-pdf-to-md.md.
