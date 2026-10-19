# Event delegation : pattern d'interactions sur conteneur

Pattern technique front réutilisable (né du composant [draw-rectangle.md](draw-rectangle.md)) — un seul listener sur un conteneur stable au lieu de N listeners sur des éléments dynamiques.

## Le pattern

```javascript
conteneur.addEventListener('pointerdown', (e) => {
  const handle = e.target.closest('[data-handle]');
  const image = e.target.closest('img');

  if (handle) {
    demarrerResize(handle.dataset.handle, handle.dataset.rectId);
    return;
  }
  if (image) {
    demarrerDrag(e);
  }
});
```

## Règles

- **Un seul handler** sur le conteneur pour tout le mode interactif — les éléments (rects, poignées) sont créés/détruits à la volée sans jamais rebrancher de listeners.
- **Ordre des discriminants** : tester les éléments SUPERPOSÉS d'abord (poignée avant image) — l'ordre du `closest()` est la logique du mode.
- **`e.currentTarget` = référence stable** (l'élément porteur du listener) → à utiliser pour `getBoundingClientRect()` et les calculs d'échelle. **`e.target` = élément réellement touché** → variable, sert à discriminer. Sur une image seule sans enfant, ils sont égaux ; ils divergent dès qu'il y a des éléments posés par-dessus.
- **Drag : listeners sur `window`** — pendant un drag, pointermove/pointerup se posent sur `window` (pas sur l'élément) : le curseur peut sortir de l'image sans casser le geste. Avec VueUse : `useEventListener(window, 'pointermove', …)` (auto-cleanup).
- **Clamp** des coordonnées aux bornes de l'image en sortie de calcul.

## Anti-patterns

- Poser un listener par élément dynamique (4 poignées × N rects).
- Calculs d'échelle sur `e.target` (variable) au lieu de `e.currentTarget` (stable).
- Pointer-events laissés à auto sur les éléments en lecture (ils interceptent les clics du mode).

## État

**vivant** — pattern acté pour le mode ciblage pdf-to-text (oct. 2026), à consolider avec le code réel du prototype.
