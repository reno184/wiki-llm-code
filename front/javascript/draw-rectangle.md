# Draw rectangle : composant Vue de ciblage de zones (drag + poignées)

Fiche technique du composant `<DrawRectangle>` du use case [pdf-to-text](../../use-cases/pdf-to-text/README.md) — surface de sélection des zones sur l'image WebP dpi300 affichée. Décisions produit : [cartographie-rect-image.md](../../use-cases/pdf-to-text/cartographie-rect-image.md).

## Trajectoire actée (oct. 2026)

- **P1 (actuel) : 100% VueUse** — pas de vanilla JS nu, pas de D3. Le drag, les listeners, la réactivité d'échelle passent par les composables VueUse.
- **P2 : composant Vue étanche** `<DrawRectangle>` — props (image source, rects existants) + emits (rects modifiés), tout le rendu interne, zéro dépendance au reste de l'app (pas de store, pas de composable métier).
- **P3 (plus tard, sur besoin réel) : D3 modulaire** — uniquement `d3-scale` + `d3-drag` (PAS la lib d3 complète), à l'intérieur du même composant, quand : dizaines de rects interactifs simultanés, aimantation/contraintes, data-join dynamique. Pattern D3-dans-Vue : le composant rend le conteneur, D3 mute tout l'intérieur, données en `shallowRef`. À documenter dans une fiche dédiée le jour venu.
- **Web Component : idée supprimée** (personne d'autre que l'app Vue ne consommera ce composant) — l'étanchéité du composant Vue (P2) suffit ; extraction WC = P4 hypothétique non planifié.
- Frontières des librairies (GSAP = motion, D3 = interactions complexes, VueUse = utilitaires) : jamais de chevauchement.

## Mécanique (décisions produit rappelées)

- **Drag rectangle** (pointerdown → pointermove → pointerup), prévisualisation temps réel ; poignées aux 4 coins pour l'ajustement fin. Rejeté : dessin par 4 clics.
- **Event delegation** : un listener sur le conteneur, discriminants par `closest()` (poignée [data-handle] testée AVANT l'image), listeners `pointermove`/`pointerup` sur `window` pendant le drag (le geste peut sortir de l'image), clamp des coordonnées aux bornes de l'image. `e.currentTarget` = référence stable pour `getBoundingClientRect()`, `e.target` = discrimination.
- **Pointer-events** : `none` sur les rects en lecture (ils n'interceptent pas les clics), `auto` sur les poignées actives uniquement.
- **États du mode** : idle → drag | resize → commit. Jamais de dessin pendant un motion (page figée — décision 6 de la fiche produit).

## VueUse : les composables du P1

- `useEventListener` — brancher `window` pointermove/pointerup pendant le drag, auto-cleanup à la fin du geste.
- `useElementSize` — réagir au redimensionnement CSS du conteneur → recalcul d'échelle dynamique (jamais supposée à 1).
- `useMouse` — si besoin de tracking continu hors geste.

## Conversion des coordonnées (rappel, détail dans [coord-scale-conversion.md](coord-scale-conversion.md))

Échelle dynamique à chaque interaction (`image.naturalWidth / rect.width`), puis `× 72/dpi` vers l'espace PDF — stockage normalisé, jamais en pixels.

## Anti-patterns

- Dessiner sur un élément en motion/transform 3D (invalide `getBoundingClientRect`).
- Supposer l'échelle d'affichage à 1.
- Importer la lib d3 complète « au cas où ».
- Poser des listeners par poignée (4N handlers) au lieu de la delegation.

## État

**en construction** — trajectoire P1→P3 actée, P1 (VueUse) à coder, consolidation de la fiche avec le code réel validé après le prototype.
