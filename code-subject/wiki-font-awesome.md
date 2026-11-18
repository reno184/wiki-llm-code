# wiki-font-awesome

Usage: fiche perso (mode-chat). **Fusion** de `wiki-fontawesome-chat.md` + `mode-agent/wiki-fontawesome-agent.md` (toutes deux supprimées nov. 2026) — contient aussi le routage agent.

## Versions accessibles

- **6.4.0 (Pro)** : téléchargeable via l'ancien compte Pro de Renaud — accès aux styles Pro (dont `fad` duotone).
- **7.3.1 (Free)** : accessible aussi.

## Setup dans un projet Vue

Arborescence à poser dans `assets/` :

```
assets/
  css/
    all.min.css
  webfonts/
```

Dans `main.css`, une seule ligne :

```css
@import "./css/all.min.css";
```

## Styles disponibles ensuite

`fa` solid, `fad` duotone (Pro), `fal` light…

## Règles agent (fusion)

- Balise `<i>` toujours. **Par défaut : solid** — si rien n'est précisé, utiliser `fa`.
- Light : `<i class="fal fa-trash"></i>`
- Exemples : `<i class="fa fa-trash"></i>`, `<i class="fal fa-trash"></i>`

## Top 15 — CRUD classique

| Icône | Classe (solid) | Usage |
|---|---|---|
| fa-plus | `<i class="fa fa-plus"></i>` | bouton Créer / Ajouter |
| fa-pen | `<i class="fa fa-pen"></i>` | Éditer (alias `fa-edit`) |
| fa-trash | `<i class="fa fa-trash"></i>` | Supprimer une ligne |
| fa-copy | `<i class="fa fa-copy"></i>` | Dupliquer |
| fa-magnifying-glass | `<i class="fa fa-magnifying-glass"></i>` | Recherche (alias `fa-search`) |
| fa-check | `<i class="fa fa-check"></i>` | Valider / confirmer |
| fa-xmark | `<i class="fa fa-xmark"></i>` | Fermer / annuler (alias `fa-times`) |
| fa-floppy-disk | `<i class="fa fa-floppy-disk"></i>` | Enregistrer (alias `fa-save`) |
| fa-eye / fa-eye-slash | `<i class="fa fa-eye"></i>` | Voir / masquer (mots de passe, détails) |
| fa-arrow-left | `<i class="fa fa-arrow-left"></i>` | Retour / navigation arrière |
| fa-arrow-right | `<i class="fa fa-arrow-right"></i>` | Suivant / ouvrir |
| fa-arrow-up + fa-arrow-down | `<i class="fa fa-arrow-up"></i>` | Tri des colonnes |
| fa-triangle-exclamation | `<i class="fa fa-triangle-exclamation"></i>` | Alerte / confirmation suppression (alias `fa-exclamation-triangle`) |
| fa-rotate | `<i class="fa fa-rotate"></i>` | Rafraîchir / rejouer (alias `fa-redo`) |
| fa-user / fa-lock | `<i class="fa fa-user"></i>` | Auth : profil utilisateur / verrou |

## Icônes fichiers & dossiers

| Icône | Classe (solid) | Usage |
|---|---|---|
| fa-file | `<i class="fa fa-file"></i>` | Fichier générique |
| fa-files | `<i class="fa fa-files"></i>` | Multi-fichiers / liste de pièces jointes |
| fa-folder / fa-folder-open | `<i class="fa fa-folder"></i>` | Dossier fermé / ouvert |
| fa-file-pdf | `<i class="fa fa-file-pdf"></i>` | PDF |
| fa-file-excel | `<i class="fa fa-file-excel"></i>` | Excel / tableur |
| fa-file-word | `<i class="fa fa-file-word"></i>` | Word |
| fa-file-csv | `<i class="fa fa-file-csv"></i>` | CSV / données tabulaires |
| fa-file-image | `<i class="fa fa-file-image"></i>` | Image |
| fa-file-zipper | `<i class="fa fa-file-zipper"></i>` | Archive (alias `fa-file-archive`) |
| fa-file-arrow-up | `<i class="fa fa-file-arrow-up"></i>` | Upload de fichier |
| fa-file-arrow-down | `<i class="fa fa-file-arrow-down"></i>` | Download de fichier |

## État

**vivant** — versions (6.4.0 Pro / 7.3.1 Free), setup Vue, styles confirmés par Renaud (oct. 2026) ; fusion fiches agent+chat et listes d'icônes CRUD/fichiers (nov. 2026).
