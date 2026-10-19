# Cartographie Rect ↔ image : ciblage des zones sur l'affichage WebP

Fiche du use case [pdf-to-text](README.md) — point 2 à cadrer. Décisions actées (oct. 2026) pour le dessin des rectangles de sélection sur l'image WebP affichée et la conversion vers les Rects PyMuPDF.

## Décisions actées

### 1. Interaction : drag + poignées de redimensionnement

- **Drag rectangle** (mousedown → mousemove → mouseup) pour créer une zone : un seul geste par zone, prévisualisation en temps réel pendant le drag.
- **Poignées aux 4 coins** du rectangle dessiné pour l'ajustement fin après création.
- Rejeté : dessin par 4 clics (4× plus d'actions, sujet aux erreurs de clic, pas de feedback en temps réel).

### 2. Échelle : jamais supposée à 1 — toujours calculée dynamiquement

L'image dpi150 est **redimensionnée par CSS** (conteneur/écran du cabinet) → l'échelle d'affichage varie. Règle : calculer `scaleX/scaleY` à chaque interaction, jamais les coder en dur.

```javascript
const rect = imageElement.getBoundingClientRect();
const scaleX = image.naturalWidth / rect.width;
const x_image = (e.clientX - rect.left) * scaleX;
```

Cette règle rend le système robuste quel que soit le conteneur CSS, le devicePixelRatio (Retina) ou le niveau de pyramide (le jour du dpi300 zoomé).

### 3. Conversion image → espace PDF (stockage normalisé)

Seconde conversion, pixels image → espace PyMuPDF (points à 72 dpi, origine **en haut à gauche**, y vers le bas) :

```javascript
const scalePdf = 72 / dpi;
const x_pdf = x_image * scalePdf;
```

- Les Rects sont **stockés en espace PDF normalisé** (source de vérité), jamais en pixels — ils survivent au changement de niveau d'affichage (150 → 300) et à la taille du conteneur.
- La conversion n'est faite qu'au moment du dessin (image → PDF) et du rendu (PDF → image, pour surligner des zones enregistrées).

### 4. Rotation des pages : hors périmètre

Décision produit : **l'avocat ne travaille pas sur des contrats scannés** → pas de gestion de `page.rotation`. Les documents avec pages tournées sont hors périmètre (si besoin un jour : la conversion Rect doit être adaptée à la rotation — piège documenté dans [infra/pymupdf.md](../../infra/pymupdf.md)).

### 5. Retour visuel de l'extraction : différé

Surimpression du texte extrait sur l'image (badge/panneau) : **à voir en second temps**. Le focus actuel = rectangles + extraction.

### 6. Jamais de dessin de rectangle pendant un motion

**Non, jamais.** Le ciblage ne se fait **jamais sur le carousel 3D en mouvement** : les transforms CSS du motion invalident le calcul simple via `getBoundingClientRect` (le rect affiché est celui de l'élément transformé, pas de l'image source) et dessiner sur un élément en rotation 3D est un cauchemar de coordonnées.

- Le ciblage se fait sur une **page figée** (pas de motion actif pendant le dessin).
- Le carousel 3D Greensock reste la vue de navigation/parcours ; le ciblage est un **mode distinct** : on fige la page, on cible, on extrait, on repart.

### 7. Le ciblage cible est le dpi300, pas le 150

Décision de périmètre pyramide :

- **dpi150** = affichage carousel/navigation (vignettes, parcours fluide).
- **dpi300** = **la surface de ciblage** — l'avocat dessine ses zones sur l'image haute résolution (lisibilité fine des clauses, précision du Rect).
- La règle d'échelle dynamique (décision 2) couvre déjà le changement de niveau : `scaleX = image.naturalWidth / rect.width` se recalcule sur l'image dpi300 affichée, et la conversion `× 72/300` vers l'espace PDF reste la même formule avec `dpi = 300`.
- Le dpi300 n'est donc plus "zoom optionnel" : c'est le **support standard du mode ciblage** (voir [affichage-webp.md](affichage-webp.md) : niveau zoom = niveau de ciblage).

### 8. Trajectoire technique du composant de ciblage

- **P1 (actuel) : 100% VueUse** — pas de vanilla nu, pas de D3 : les composables VueUse (`useEventListener`, `useElementSize`) portent le drag et la réactivité d'échelle.
- **P2 : composant Vue étanche** `<DrawRectangle>` — props + emits, tout le rendu interne, zéro dépendance au reste de l'app.
- **P3 (plus tard, sur besoin réel) : D3 modulaire** — uniquement `d3-scale` + `d3-drag` (pas la lib complète), quand les interactions deviennent complexes (dizaines de rects, contraintes, data-join).
- **Web Component : idée supprimée** — personne d'autre que l'app Vue ne consommera le composant ; l'étanchéité P2 suffit.
- Fiches techniques : [draw-rectangle.md](../../front/javascript/draw-rectangle.md), [event-delegation.md](../../front/javascript/event-delegation.md), [coord-scale-conversion.md](../../front/javascript/coord-scale-conversion.md) — ce sont ELLES que les agents de coding chargent, pas cette fiche produit.

## Chaîne complète (validée)

```
Drag sur l'image affichée (clientX/Y + getBoundingClientRect + naturalWidth)
  → coords pixels image (scaleX/scaleY dynamiques)
  → coords espace PDF (× 72/dpi) — stockage normalisé
  → POST API {docId, page, rect: [x0, y0, x1, y1]}
  → micro-service : pymupdf.Rect(x0, y0, x1, y1) + page.get_text(clip=cr)
  → texte extrait retourné au front
```

## Questions restantes (à trancher plus tard — prototype en cours, priorité étapes 1-3)

1. Contrat d'API exact : format de la requête (un rect par requête, ou lot de rects par document ?), réponse (texte brut ? avec numéros de page ?).
2. Stockage des zones : en base avec le blurhash (métadonnées du document), historique des zones dessinées par document ?
3. Comportement multi-zones : zone unique active, ou plusieurs zones simultanées avec sélection/cumul des textes extraits ?

## État

**en construction** — décisions 1-8 actées (drag+poignées, échelle dynamique, conversion normalisée, rotation hors périmètre, retour visuel différé, jamais de dessin pendant un motion/page figée, ciblage sur dpi300, trajectoire P1 VueUse → P2 composant étanche → P3 D3 modulaire, WC supprimé) ; questions 1-3 (contrat d'API, stockage, multi-zones) à trancher plus tard.
