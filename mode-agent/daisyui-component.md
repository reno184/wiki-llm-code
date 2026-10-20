# daisyui-component — DaisyUI

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

`<span class="loading loading-infinity loading-xl"></span>` — pour les requêtes vers les routes qui utilisent `invoke` LangChain (traitement long).
