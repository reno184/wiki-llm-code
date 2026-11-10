# inspiration-html-css — Mises en page à réutiliser

> Exemples de mises en page HTML/CSS (daisyUI/Tailwind) prêtes à copier-coller et à adapter.

## formulaire

### footer de formulaire — boutons alignés à droite

Boutons d'action en fin de formulaire, alignés à droite (`flex justify-end`), Reset discret + submit :

```html
<footer class="flex justify-end items-center">
  <button class="btn btn-xs btn-ghost me-2 " type="reset">Reset</button>
  <input type="submit" class="btn btn-sm btn-neutral "/>
</footer>
```

- `me-2` (margin-end) espace les deux boutons ; adapter les variantes (`btn-ghost`, `btn-neutral`) au thème.
- `input type="submit"` plutôt que `<button>` : soumet le formulaire sans JS supplémentaire.

## Fiches liées

- [../front/vuejs-form.md](../front/vuejs-form.md) — contrôles de formulaire Vue + daisyUI (select required + option placeholder via v-model).
- [../front/daisyui-components.md](../front/daisyui-components.md) — composants daisyUI.
- [../mode-agent/daisyui-component.md](../mode-agent/daisyui-component.md) — version agent (btn loading, input file masqué).
- [code-subject/wiki-tailwindcss.md](code-subject/wiki-tailwindcss.md) — utilitaires Tailwind.
