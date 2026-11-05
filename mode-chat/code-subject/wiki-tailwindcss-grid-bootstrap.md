# Tailwind CSS — grid responsive façon Bootstrap (row / col-N / col-{sm,md,lg,xl,xxl}-N)

## Problème

Refaire un projet en Tailwind mais garder le réflexe Bootstrap : une `row`, des colonnes `col-6`, `col-4`, `col-3` qui passent responsive par préfixe (`col-md-3`…). Écrire `grid grid-cols-12` + `col-span-*` à la main partout est verbeux et ne suit pas la logique mobile-first de Bootstrap.

## Solution : couche CSS utilitaire à coller une fois

Coller ce bloc dans le CSS global du projet (ex. `src/assets/main.css`), éventuellement dans `@layer components` pour laisser les utilities Tailwind gagner en spécificité.

Breakpoints alignés sur les défauts Tailwind : `sm` 640px, `md` 768px, `lg` 1024px, `xl` 1280px, `xxl` 1536px.

```css
.row {
  display: grid;
  grid-template-columns: repeat(12, minmax(0, 1fr));
  gap: 1.5rem;
}

[class*="col-"] {
  min-width: 0;
}

.col-12 { grid-column: span 12; }
.col-6  { grid-column: span 6; }
.col-4  { grid-column: span 4; }
.col-3  { grid-column: span 3; }

@media (min-width: 640px) {
  .col-sm-12 { grid-column: span 12; }
  .col-sm-6  { grid-column: span 6; }
  .col-sm-4  { grid-column: span 4; }
  .col-sm-3  { grid-column: span 3; }
}

@media (min-width: 768px) {
  .col-md-12 { grid-column: span 12; }
  .col-md-6  { grid-column: span 6; }
  .col-md-4  { grid-column: span 4; }
  .col-md-3  { grid-column: span 3; }
}

@media (min-width: 1024px) {
  .col-lg-12 { grid-column: span 12; }
  .col-lg-6  { grid-column: span 6; }
  .col-lg-4  { grid-column: span 4; }
  .col-lg-3  { grid-column: span 3; }
}

@media (min-width: 1280px) {
  .col-xl-12 { grid-column: span 12; }
  .col-xl-6  { grid-column: span 6; }
  .col-xl-4  { grid-column: span 4; }
  .col-xl-3  { grid-column: span 3; }
}

@media (min-width: 1536px) {
  .col-xxl-12 { grid-column: span 12; }
  .col-xxl-6  { grid-column: span 6; }
  .col-xxl-4  { grid-column: span 4; }
  .col-xxl-3  { grid-column: span 3; }
}
```

## Usage — même logique mobile-first que Bootstrap

```html
<div class="row">
  <div class="col-12 col-md-6 col-lg-3">1 colonne mobile, 2 tablette, 4 desktop</div>
  <div class="col-12 col-md-6 col-lg-3">…</div>
</div>
```

Classe sans préfixe = comportement mobile ; `col-md-*` écrase dès 768px, etc.

## Checklist « ne rien oublier »

- `minmax(0, 1fr)` obligatoire : avec `1fr` seul, un contenu large (table, image sans max-width) fait déborder la colonne et pousse les voisines.
- `[class*="col-"] { min-width: 0 }` : même raison, empêche le dépassement du contenu.
- Gutter : `gap: 1.5rem` ≈ g-4 Bootstrap ; adapter à la maquette, pas de marges négatives nécessaires (grid ≠ flex).
- Utiliser la même nomenclature de breakpoints que le reste du projet Tailwind, sinon deux échelles cohabitent (ex. Bootstrap `lg` = 992px ≠ Tailwind `lg` = 1024px).
- Le scanner Tailwind ne purge pas ces classes : c'est du CSS classique, pas des utilities générées — aucune safelist nécessaire.

## Liens croisés

- [wiki-tailwindcss.md](wiki-tailwindcss.md) — l'autre piège Tailwind : classes dynamiques interpolées invisibles au build (safelist)
- [wiki-vuejs-reset.md](wiki-vuejs-reset.md) — shell projet Vue où coller ce bloc CSS
