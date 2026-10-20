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


