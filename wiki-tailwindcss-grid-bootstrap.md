# Tailwind CSS v4 — reproduire la grid Bootstrap (row / col-12 / col-md-3) avec les classes natives

## Contexte

La reproduction de Bootstrap avec les classes `row`, `col-12`, `col-md-3` ne fonctionne pas directement : le projet utilise **Tailwind CSS v4** et **DaisyUI v5** (pas la bibliothèque Bootstrap CSS). Les classes Bootstrap n'existent pas — il faut les traduire en classes Tailwind natives.

## Solution 1 : Flexbox (recommandée)

Au lieu de `row` + `col-12 col-md-3` :

```vue
<template>
  <main>
    <div>SearchWordUploadByTag</div>
    <header class="flex flex-wrap -mx-2">
      <div class="w-full md:w-1/4 px-2">

      </div>
      <div class="w-full md:w-1/4 px-2">
        <TagForm :tags="tags" :onRemove="handleRemove" :onAdd="handleAdd"/>
      </div>
    </header>
  </main>
</template>
```

## Solution 2 : CSS Grid

Alternative très propre en Tailwind pour créer des colonnes :

```vue
<template>
  <main>
    <div>SearchWordUploadByTag</div>
    <header class="grid grid-cols-1 md:grid-cols-4 gap-4">
      <div>

      </div>
      <div>
        <TagForm :tags="tags" :onRemove="handleRemove" :onAdd="handleAdd"/>
      </div>
    </header>
  </main>
</template>
```

## Correspondances Bootstrap ↔ Tailwind CSS

| Bootstrap | Tailwind CSS équivalent |
| :--- | :--- |
| `row` | `flex flex-wrap` ou `grid grid-cols-12` |
| `col-12` | `w-full` (ou `col-span-12`) |
| `col-6` | `w-1/2` (ou `col-span-6`) |
| `col-4` | `w-1/3` (ou `col-span-4`) |
| `col-3` | `w-1/4` (ou `col-span-3`) |
| `col-md-3` | `md:w-1/4` (ou `md:col-span-3`) |
| `col-lg-4` | `lg:w-1/3` (ou `lg:col-span-4`) |

## Exemple appliqué au fichier (SearchWordUploadByTagPage.vue)

```vue
<template>
  <main>
    <div>SearchWordUploadByTag</div>
    <header class="grid grid-cols-1 md:grid-cols-4 gap-4">
      <div>

      </div>
      <div>
        <TagForm :tags="tags" :onRemove="handleRemove" :onAdd="handleAdd"/>
      </div>
    </header>
  </main>
</template>

<script setup>
// ... existing code ...
</script>
```

## Checklist « ne rien oublier »

- Toujours un `grid-cols-1` de base en mobile-first, puis `md:grid-cols-4` — jamais seulement `grid-cols-4` sinon 4 colonnes sur mobile.
- Flexbox : les gutters se font avec `px-2` sur les enfants + `-mx-2` sur le parent (marges négatives), l'équivalent des `g-*` Bootstrap.
- Grid : le `gap-4` remplace les gutters, pas de marges négatives nécessaires.
- Préfixes Tailwind = mêmes réflexes que Bootstrap : `sm` 640px, `md` 768px, `lg` 1024px, `xl` 1280px (⚠️ ≠ Bootstrap : `lg` = 992px chez Bootstrap).
- DaisyUI v5 ne fournit pas de composant grid : utiliser les utilities Tailwind natives.

## Liens croisés

- [wiki-tailwindcss.md](wiki-tailwindcss.md) — l'autre piège Tailwind : classes dynamiques interpolées invisibles au build (safelist)
- [wiki-vuejs-reset.md](wiki-vuejs-reset.md) — shell projet Vue (Tailwind v4 + DaisyUI v5) où appliquer ces classes
