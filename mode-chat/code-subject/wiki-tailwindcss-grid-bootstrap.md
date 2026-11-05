# Tailwind CSS — grid responsive façon Bootstrap (row, col-N, offset, d-none, col-{sm,md,lg,xl,xxl}-N)

## Problème

Refaire un projet en Tailwind mais garder le réflexe Bootstrap : une `row`, des colonnes `col-6`, `col-4`, `col-3`, des offsets `offset-md-3` et du masquage responsive `d-none d-md-block`. Écrire `grid grid-cols-12` + `col-span-*` à la main partout est verbeux et ne suit pas la logique mobile-first de Bootstrap.

## Solution : couche CSS utilitaire à coller une fois

Coller ce bloc dans le CSS global du projet (ex. `src/assets/main.css`). `@layer components` pour laisser les utilities Tailwind gagner en spécificité (optionnel — retirer le wrapper si le projet n'utilise pas les layers).

Breakpoints alignés sur les défauts Tailwind : `sm` 640px, `md` 768px, `lg` 1024px, `xl` 1280px, `xxl` 1536px.

```css
@layer components {
  .row {
    display: grid;
    grid-template-columns: repeat(12, minmax(0, 1fr));
    gap: 1.5rem;
  }

  [class*="col-"],
  [class*="offset-"] {
    min-width: 0;
  }

  .col-12 { grid-column: span 12; }
  .col-6 { grid-column: span 6; }
  .col-4 { grid-column: span 4; }
  .col-3 { grid-column: span 3; }
  .offset-0 { margin-left: 0; }
  .offset-1 { margin-left: 8.3333%; }
  .offset-2 { margin-left: 16.6667%; }
  .offset-3 { margin-left: 25%; }
  .offset-4 { margin-left: 33.3333%; }
  .offset-5 { margin-left: 41.6667%; }
  .offset-6 { margin-left: 50%; }
  .offset-7 { margin-left: 58.3333%; }
  .offset-8 { margin-left: 66.6667%; }
  .offset-9 { margin-left: 75%; }
  .offset-10 { margin-left: 83.3333%; }
  .offset-11 { margin-left: 91.6667%; }

  @media (min-width: 640px) {
    .col-sm-12 { grid-column: span 12; }
    .col-sm-6 { grid-column: span 6; }
    .col-sm-4 { grid-column: span 4; }
    .col-sm-3 { grid-column: span 3; }
    .offset-sm-0 { margin-left: 0; }
    .offset-sm-1 { margin-left: 8.3333%; }
    .offset-sm-2 { margin-left: 16.6667%; }
    .offset-sm-3 { margin-left: 25%; }
    .offset-sm-4 { margin-left: 33.3333%; }
    .offset-sm-5 { margin-left: 41.6667%; }
    .offset-sm-6 { margin-left: 50%; }
    .offset-sm-7 { margin-left: 58.3333%; }
    .offset-sm-8 { margin-left: 66.6667%; }
    .offset-sm-9 { margin-left: 75%; }
    .offset-sm-10 { margin-left: 83.3333%; }
    .offset-sm-11 { margin-left: 91.6667%; }
    .d-sm-none { display: none; }
    .d-sm-block { display: block; }
    .d-sm-inline { display: inline; }
    .d-sm-inline-block { display: inline-block; }
    .d-sm-flex { display: flex; }
    .d-sm-inline-flex { display: inline-flex; }
    .d-sm-grid { display: grid; }
  }

  @media (min-width: 768px) {
    .col-md-12 { grid-column: span 12; }
    .col-md-6 { grid-column: span 6; }
    .col-md-4 { grid-column: span 4; }
    .col-md-3 { grid-column: span 3; }
    .offset-md-0 { margin-left: 0; }
    .offset-md-1 { margin-left: 8.3333%; }
    .offset-md-2 { margin-left: 16.6667%; }
    .offset-md-3 { margin-left: 25%; }
    .offset-md-4 { margin-left: 33.3333%; }
    .offset-md-5 { margin-left: 41.6667%; }
    .offset-md-6 { margin-left: 50%; }
    .offset-md-7 { margin-left: 58.3333%; }
    .offset-md-8 { margin-left: 66.6667%; }
    .offset-md-9 { margin-left: 75%; }
    .offset-md-10 { margin-left: 83.3333%; }
    .offset-md-11 { margin-left: 91.6667%; }
    .d-md-none { display: none; }
    .d-md-block { display: block; }
    .d-md-inline { display: inline; }
    .d-md-inline-block { display: inline-block; }
    .d-md-flex { display: flex; }
    .d-md-inline-flex { display: inline-flex; }
    .d-md-grid { display: grid; }
  }

  @media (min-width: 1024px) {
    .col-lg-12 { grid-column: span 12; }
    .col-lg-6 { grid-column: span 6; }
    .col-lg-4 { grid-column: span 4; }
    .col-lg-3 { grid-column: span 3; }
    .offset-lg-0 { margin-left: 0; }
    .offset-lg-1 { margin-left: 8.3333%; }
    .offset-lg-2 { margin-left: 16.6667%; }
    .offset-lg-3 { margin-left: 25%; }
    .offset-lg-4 { margin-left: 33.3333%; }
    .offset-lg-5 { margin-left: 41.6667%; }
    .offset-lg-6 { margin-left: 50%; }
    .offset-lg-7 { margin-left: 58.3333%; }
    .offset-lg-8 { margin-left: 66.6667%; }
    .offset-lg-9 { margin-left: 75%; }
    .offset-lg-10 { margin-left: 83.3333%; }
    .offset-lg-11 { margin-left: 91.6667%; }
    .d-lg-none { display: none; }
    .d-lg-block { display: block; }
    .d-lg-inline { display: inline; }
    .d-lg-inline-block { display: inline-block; }
    .d-lg-flex { display: flex; }
    .d-lg-inline-flex { display: inline-flex; }
    .d-lg-grid { display: grid; }
  }

  @media (min-width: 1280px) {
    .col-xl-12 { grid-column: span 12; }
    .col-xl-6 { grid-column: span 6; }
    .col-xl-4 { grid-column: span 4; }
    .col-xl-3 { grid-column: span 3; }
    .offset-xl-0 { margin-left: 0; }
    .offset-xl-1 { margin-left: 8.3333%; }
    .offset-xl-2 { margin-left: 16.6667%; }
    .offset-xl-3 { margin-left: 25%; }
    .offset-xl-4 { margin-left: 33.3333%; }
    .offset-xl-5 { margin-left: 41.6667%; }
    .offset-xl-6 { margin-left: 50%; }
    .offset-xl-7 { margin-left: 58.3333%; }
    .offset-xl-8 { margin-left: 66.6667%; }
    .offset-xl-9 { margin-left: 75%; }
    .offset-xl-10 { margin-left: 83.3333%; }
    .offset-xl-11 { margin-left: 91.6667%; }
    .d-xl-none { display: none; }
    .d-xl-block { display: block; }
    .d-xl-inline { display: inline; }
    .d-xl-inline-block { display: inline-block; }
    .d-xl-flex { display: flex; }
    .d-xl-inline-flex { display: inline-flex; }
    .d-xl-grid { display: grid; }
  }

  @media (min-width: 1536px) {
    .col-xxl-12 { grid-column: span 12; }
    .col-xxl-6 { grid-column: span 6; }
    .col-xxl-4 { grid-column: span 4; }
    .col-xxl-3 { grid-column: span 3; }
    .offset-xxl-0 { margin-left: 0; }
    .offset-xxl-1 { margin-left: 8.3333%; }
    .offset-xxl-2 { margin-left: 16.6667%; }
    .offset-xxl-3 { margin-left: 25%; }
    .offset-xxl-4 { margin-left: 33.3333%; }
    .offset-xxl-5 { margin-left: 41.6667%; }
    .offset-xxl-6 { margin-left: 50%; }
    .offset-xxl-7 { margin-left: 58.3333%; }
    .offset-xxl-8 { margin-left: 66.6667%; }
    .offset-xxl-9 { margin-left: 75%; }
    .offset-xxl-10 { margin-left: 83.3333%; }
    .offset-xxl-11 { margin-left: 91.6667%; }
    .d-xxl-none { display: none; }
    .d-xxl-block { display: block; }
    .d-xxl-inline { display: inline; }
    .d-xxl-inline-block { display: inline-block; }
    .d-xxl-flex { display: flex; }
    .d-xxl-inline-flex { display: inline-flex; }
    .d-xxl-grid { display: grid; }
  }

  .d-none { display: none; }
  .d-block { display: block; }
  .d-flex { display: flex; }
}
```

## Usage — même logique mobile-first que Bootstrap

Colonnes :

```html
<div class="row">
  <div class="col-12 col-md-6 col-lg-3">1 colonne mobile, 2 tablette, 4 desktop</div>
  <div class="col-12 col-md-6 col-lg-3">…</div>
</div>
```

Offset (colonne de 6 centrée) :

```html
<div class="row">
  <div class="col-12 col-md-6 offset-md-3">centrée à partir de md</div>
</div>
```

Masquage responsive (caché sur mobile, visible dès md) :

```html
<aside class="d-none d-md-block col-md-3">menu caché mobile</aside>
<div class="col-12 col-md-9">contenu</div>
```

Classe sans préfixe = comportement mobile ; `col-md-*`/`offset-md-*`/`d-md-*` écrasent dès 768px, etc. Ordre dans le HTML indifférent : c'est le poids des media queries qui décide (plus grand breakpoint gagne).

## Équivalences directes avec les utilities Tailwind natives

| Bootstrap-like | Tailwind natif |
|---|---|
| `d-none d-md-block` | `hidden md:block` |
| `d-block d-lg-none` | `block lg:hidden` |
| `offset-md-3` | `md:col-start-5` (ou `md:ml-1/4`) |
| `col-md-6` | `md:col-span-6` (si `grid-cols-12` sur le parent) |

Les deux systèmes cohabitent : le CSS maison est du CSS classique, jamais purgé par le scanner Tailwind.

## Checklist « ne rien oublier »

- `minmax(0, 1fr)` obligatoire : avec `1fr` seul, un contenu large (table, image sans max-width) fait déborder la colonne et pousse les voisines.
- `[class*="col-"], [class*="offset-"] { min-width: 0 }` : même raison, empêche le dépassement du contenu.
- Offset = `margin-left` en pourcentage (comme Bootstrap) : fonctionne sur grid, se combine avec `col-*` sans conflit `grid-column`.
- `offset-*` supérieur au nombre de colonnes restantes → la colonne déborde sur une nouvelle ligne de grid. Vérifier offset + largeur ≤ 12.
- Gutter : `gap: 1.5rem` ≈ g-4 Bootstrap ; adapter à la maquette, pas de marges négatives nécessaires (grid ≠ flex).
- Utiliser la même nomenclature de breakpoints que le reste du projet Tailwind, sinon deux échelles cohabitent (ex. Bootstrap `lg` = 992px ≠ Tailwind `lg` = 1024px).
- Le scanner Tailwind ne purge pas ces classes : c'est du CSS classique, pas des utilities générées — aucune safelist nécessaire.

## Liens croisés

- [wiki-tailwindcss.md](wiki-tailwindcss.md) — l'autre piège Tailwind : classes dynamiques interpolées invisibles au build (safelist)
- [wiki-vuejs-reset.md](wiki-vuejs-reset.md) — shell projet Vue où coller ce bloc CSS
