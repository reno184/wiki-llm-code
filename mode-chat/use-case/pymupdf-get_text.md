# PyMuPDF : extraction de texte

## Les trois modes de lecture du texte

### 1. `page.get_text("blocks", sort=True)`

- Cas d'usage : récupérer des paragraphes entiers dans l'ordre de lecture, ou repérer des sections par leur zone géométrique.
- Extraction « macro », moins fine que `get_text("dic")`.
- Exemple : un paragraphe, une image, un paragraphe → 3 blocks. Un paragraphe, un tableau, un paragraphe → 3 blocks également.

### 2. `page.get_text("dic")`

Dictionnaire hiérarchique `page → blocks → lines → spans`. C'est la source de tout : chaque niveau porte ses coordonnées, et les **spans** portent le style (police, taille, gras, couleur).

- C'est ce qui permet de distinguer un **titre** (grande taille / gras) d'un paragraphe.

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

- Piège : filtrer les blocks d'images (`type != 0`), sinon `KeyError` sur `block["lines"]`.

### 4. `page.get_text("words")`

Granularité maximale : une liste de mots avec leurs coordonnées `(x0, y0, x1, y1, mot, ...)`. `sort=True` est disponible.

## Ciblage géométrique

### 5. `clip` + `pymupdf.Rect(rx0, ry0, rx1, ry1)`

`Rect` est un rectangle en coordonnées page, avec l'origine **en haut à gauche** et l'axe y orienté vers le bas.

```python
cr = pymupdf.Rect(50, 100, 300, 200)
texte_zone = page.get_text(clip=cr)
```

### 6. `page.search_for("Répondre aux questions")`

Retourne une **liste de Rects** (un par occurrence).
En cherchant un mot du **début** d'une zone avec `search_for("debut")`, puis un mot de la **fin** avec `search_for("fin")`, on obtient deux Rects qui délimitent la zone. On en déduit le Rect englobant, puis on extrait le texte qu'il contient avec `page.get_text(clip=cr)`.

## Images

`page.get_images(full=True)` liste les images référencées du document (xref, dimensions, compression), mais **sans position**.

`doc.extract_image(xref)` extrait l'image brute (bytes + extension).

`get_image_rects(xref)` / `get_image_info()` donnent les **positions** des images sur la page — l'équivalent de `search_for` pour les images. Indispensable pour associer chaque image à la bonne section de texte.

- `page.get_pixmap(dpi=150)` → rendu de la page **entière** en image (PNG via `pix.save()` / `pix.tobytes()`) : utile pour l'OCR ou pour donner la page à un LLM vision (use-case ocr-llm : base64 → ollama).
- `page.get_pixmap(clip=cr, dpi=200)` → rendu **d'une zone seulement** : la leçon de la fiche ocr-llm est de découper les zones denses avant envoi au modèle.

Cas d'usage : OCR d'un schéma dans un PDF ; vérifier **visuellement** si toutes les pages d'un document portent une signature (détection par rendu d'image, sans vérification cryptographique).

## Objets vectoriels et structure

### 10. `page.get_drawings()` + `cluster_drawings()`

Tous les objets vectoriels (lignes, rectangles, courbes) avec leurs couleurs et épaisseurs. Permet de détecter des **tableaux sans bordures** (lignes de séparation), des surlignages, des cadres. `cluster_drawings()` regroupe les dessins proches (zones de tableau).

### 11. `doc.get_toc()`

La **table des matières** du PDF (bookmarks) : une hiérarchie de titres obtenue gratuitement, sans avoir à la deviner via les tailles de police. À croiser avec les spans du `dic` (détection par font-size) pour fiabiliser la détection des titres.

Un PDF généré par Word a souvent une table des matières accessible via `get_toc`.

Cas d'usage : si on demande à un LLM de reconstituer un Markdown, lui fournir la table des matières en amont l'aide.

## Tableaux et liens

### 12. `page.find_tables()`

Détecte les tableaux (par bordures ou par alignements).

- `page.find_tables().tables` → liste ; `table.extract()` → liste de listes (lignes × colonnes), prêt pour CSV/pandas.
- `table.bbox` pour les coordonnées, `table.header` pour l'en-tête.

- Piège : la détection n'est pas infaillible (sans bordures, cellules fusionnées) → vérifier visuellement sur un échantillon. Alternative pour les tableaux **sans bordures (lignes vectorielles)** : `get_drawings()` (point 10).

### 13. `page.get_links()`

Liste les liens de la page : chaque lien porte un `kind` (LINK_URI, LINK_GOTO…), un `from` (Rect cliquable) et sa cible (`uri` ou `page`). Indispensable pour ne pas perdre les liens lors d'une conversion Markdown.

## Autres

- **`page.annots()`** — annotations (surlignages, notes) : un surlignage = Rect + couleur, souvent porteur de sens. Use-case : un avocat surligne ce qu'il veut anonymiser, on repère les zones marquées.

## Métadonnées

- **`doc.metadata`** — auteur, titre, dates.
- **Point vérifié : le stockage de règles d'anonymisation dans les métadonnées est faisable.** `doc.set_metadata()` est limité aux clés standard, mais deux voies permettent d'embarquer des clés personnalisées dans le PDF lui-même :
  - le dictionnaire `/Info` en bas niveau via `doc.xref_set_key()` / `doc.xref_get_key()` ;
  - le flux XMP via `doc.set_xml_metadata()` / `doc.get_xml_metadata()` (mieux adapté à des règles structurées en XML).
- Hyper pratique : le pipeline d'anonymisation peut ainsi lire les règles directement depuis le PDF, sans configuration externe.

## Anonymisation

PyMuPDF permet de manipuler directement les PDF : dessiner sur le document, insérer du texte, ou supprimer du texte existant. Grâce à `search_for`, on peut localiser automatiquement des zones sensibles et réaliser une anonymisation de PDF sans passer par un outil externe.

## Use-cases possibles

- vérifier qu'un PDF est bien signé ;
- reconstruire un PDF anonymisé ;
- faire comprendre la sémantique d'un PDF pour un envoi automatique au process de wiki-llm ;
- détecter des tableaux et extraire en masse des informations vers Excel — par exemple, sur 500 fiches de paie, retrouver le tableau des congés payés et produire un CSV.

## État

**Vivant**