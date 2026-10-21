# daisyui-component — DaisyUI

> Utiliser en priorité daisyUi avant des classes natives tailwindcss

Usage: snippets composants DaisyUI.

## btn loading

À utiliser sur `isFetching` du composable `useApi` :

```html
<button class="btn">
  <span class="loading loading-spinner"></span>
  loading
</button>
```

## loader traitement long (LangChain invoke)

A utiliser pour les requêtes vers les router backend qui utilisent `invoke` LangChain (traitement long).
`<span class="loading loading-infinity"></span>`



## input file masqué — label btn (à la place de file-input)

Ne pas utiliser le `<input type="file">` affiché de daisyUI (`file-input`) :

```html
<input type="file" name="file" accept="image/png" required class="file-input file-input-bordered file-input-primary w-full"/>
```

Utiliser à la place un **label stylé btn** contenant l'input masqué (`class="hidden"`) :

```html
<label class="btn btn-olive cursor-pointer">
  <span>Sélectionner un fichier</span>
  <input type="file" class="hidden" accept=".pdf,application/pdf" @change="e => onChange(e.target.files)"/>
</label>
```

- Le `file-input` natif de daisyUI est moche ; le label btn donne un vrai bouton, l'input est invisible mais fonctionnel (clic sur le label = ouverture du sélecteur de fichiers).
- Adapter `btn-olive` et `accept` au projet.

## carousel full width — défiler des groupes d'images (ex. pages WebP d'un upload)

Parfait pour faire défiler des images WebP une par une (pages d'un PDF converti, groupe d'images) :

```html
<div class="carousel rounded-box w-64">
  <div class="carousel-item w-full">
    <img
      src="https://img.daisyui.com/images/stock/photo-1559703248-dcaaec9fab78.webp"
      class="w-full"
      alt="Tailwind CSS Carousel component" />
  </div>
  <div class="carousel-item w-full">
    <img
      src="https://img.daisyui.com/images/stock/photo-1565098772267-60af42b81ef2.webp"
      class="w-full"
      alt="Tailwind CSS Carousel component" />
  </div>
</div>
```

- `carousel-item w-full` = un seul item visible à la fois, scroll horizontal (drag/touch ou flèches clavier) entre les items ; `w-64` sur le conteneur = largeur du carousel.
- En Vue : boucle `v-for` sur la liste des images (ex. `v-for="page in group.pages"`) avec le `src` de chaque WebP — les divs `carousel-item` sont générés dynamiquement.
- Pas de JS nécessaire : le scroll natif fait défiler ; pour un contrôle programmatique, viser l'élément avec `scrollIntoView`.
