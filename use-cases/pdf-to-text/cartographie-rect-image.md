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

## Chaîne complète (validée)

```
Drag sur l'image affichée (clientX/Y + getBoundingClientRect + naturalWidth)
  → coords pixels image (scaleX/scaleY dynamiques)
  → coords espace PDF (× 72/dpi) — stockage normalisé
  → POST API {docId, page, rect: [x0, y0, x1, y1]}
  → micro-service : pymupdf.Rect(x0, y0, x1, y1) + page.get_text(clip=cr)
  → texte extrait retourné au front
```

## Questions restantes (à trancher)

1. Contrat d'API exact : format de la requête (un rect par requête, ou lot de rects par document ?), réponse (texte brut ? avec numéros de page ?).
2. Stockage des zones : en base avec le blurhash (métadonnées du document), historique des zones dessinées par document ?
3. Comportement multi-zones : zone unique active, ou plusieurs zones simultanées avec sélection/cumul des textes extraits ?
4. Z-index du canvas de sélection par rapport au carousel 3D Greensock (dessine-t-on sur le carousel en mouvement ou en vue page fixe ?).
5. dpi300 zoomé : le ciblage se fait-il aussi sur l'image zoomée (donc sur dpi300, échelle recalculée — déjà couvert par la règle dynamique) ?

## État

**en construction** — décisions 1-5 actées (drag+poignées, échelle dynamique, conversion normalisée, rotation hors périmètre, retour visuel différé) ; questions restantes à trancher avant le codage.
