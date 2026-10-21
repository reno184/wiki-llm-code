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
