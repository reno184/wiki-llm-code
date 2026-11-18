# Anonymisation PDF : le rectangle de rédaction supprime réellement le texte

> Use-case : anonymiser des zones d'un PDF via PyMuPDF. Réponse à la question « le rectangle noir supprime-t-il vraiment le texte, ou juste le masque ? ».

## TL;DR

Oui : la rédaction PyMuPDF (`add_redact_annot` + `apply_redactions`) **supprime physiquement le texte du fichier PDF**. Ce n'est pas un masque ni un rectangle noir dessiné par-dessus. Une fois le fichier enregistré, le texte est irrécupérable.

À distinguer de `page.draw_rect(...)` / `shape.draw_rect` : là, on ajoute une couche graphique AU-DESSUS du texte — il reste sélectionnable, copiable et extractible. Voir [pymupdf_tech_anonymisation.md](pymupdf_tech_anonymisation.md) pour les recettes complètes (masquage, suppression, remplacement).

## Les 2 étapes indispensables

### Étape 1 — annotation d'intention (`add_redact_annot`)

```python
rect = pymupdf.Rect(x0, y0, x1, y1) * page.derotation_matrix
page.add_redact_annot(rect, fill=(0, 0, 0))
```

- `add_redact_annot` ajoute une **annotation d'intention de rédaction** sur la zone (rien n'est encore supprimé).
- `fill=(0, 0, 0)` : couleur du rectangle posé après application (ici noir).
- `* page.derotation_matrix` : corrige les coordonnées si la page est tournée/pivotée.

### Étape 2 — application effective (`apply_redactions`)

Dans `redact_pdf_zones` de `service.py`, juste après la boucle d'ajout :

```python
for page in doc:
    page.apply_redactions()
```

Sans `apply_redactions()`, **rien n'est supprimé** — les annotations restent de simples intentions.

## Ce que fait concrètement `apply_redactions()`

1. **Suppression du texte vectoriel** : caractères, glyphes et flux de données textuelles dans `rect` retirés de la structure du PDF → plus sélectionnable, plus copiable, plus lisible par extraction ou lecteur d'écran.
2. **Découpage des images** : si une image est dans la zone, ses pixels sont effacés/remplacés.
3. **Rendu visuel** : la zone est recouverte de la couleur `fill`.

## Pourquoi c'est définitif

- Modifie la structure interne des objets de la page.
- Supprime les instructions d'affichage du texte dans le cadre `rect`.
- À l'enregistrement : `doc.tobytes(garbage=3, deflate=True)` nettoie les objets orphelins/non référencés.

**Résultat : le texte n'est plus présent dans le PDF final et ne peut pas être récupéré.**

## Pièges

- Oublier `apply_redactions()` → simple annotation, texte intact.
- Confondre avec `draw_rect` → masque cosmétique, texte toujours extractible.
- Enregistrer avec `garbage=3` pour purger les résidus d'objets.
