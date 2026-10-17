# DaisyUI — composants et classes utilisés

Stack: TailwindCSS + DaisyUI, Vue 3 (classes applicables partout).
Usage: inventaire des classes DaisyUI réellement utilisées dans les projets du wiki — pour qu'un agent génère du markup avec **les classes DaisyUI du wiki**, pas des classes Tailwind brutes ni des composants Bootstrap imaginaires. Le principe : DaisyUI fournit des composants prêts à l'emploi (une classe = un composant stylé), Tailwind complète avec les utilitaires (spacing, layout, couleurs).

## Règles

- **Toujours des classes DaisyUI pour les composants** (boutons, inputs, cards, menu...) : `btn`, `input`, `select`, `card`, `menu`, `modal`, `alert`... Un agent ne recode pas un bouton avec `border rounded px-4` quand `btn btn-primary` existe.
- Les **modificateurs** se combinent : `btn btn-primary btn-sm`, `file-input file-input-primary`, `alert alert-error`.
- Tailwind pour tout le reste : `w-full`, `flex`, gap, responsive `md:`, etc.
- Ne pas mélanger : un composant DaisyUI + les utilitaires Tailwind autour, jamais de styles custom dupliquant DaisyUI.

## Composants utilisés

### `file-input` — input upload de fichier

```html
<div class="form-control">
    <input type="file" class="file-input file-input-primary w-full" accept=".pdf,application/pdf" @change="handleChange">
</div>
```

- `file-input` : le composant input file stylé DaisyUI.
- `file-input-primary` (ou `file-input-secondary`, etc.) : la couleur.
- `w-full` : utilitaire Tailwind pour la largeur.
- Voir [../front/upload-file.md](../front/upload-file.md) pour le `handleChange` et le contrat single/`multiple`.

### `menu` — menu horizontal de navigation

```html
<ul class="menu menu-horizontal rounded-box">
    <li><router-link :to="menu.path">...</router-link></li>
</ul>
```

- `menu` : le composant navigation (verticalement par défaut), `menu-horizontal` : en barre horizontale.
- L'état actif passe par `linkActiveClass: 'menu-active'` dans vue-router — voir [../front/vuejs-router-menu.md](../front/vuejs-router-menu.md).

## À documenter au fil des projets

Composants DaisyUI croisés dans de nouveaux projets → ajouter une section ici avec le markup type et les pièges (boutons `btn`, toasts, modals, alerts, tables, badges, loading...).

## État

**vivant** — inventaire évolutif des composants DaisyUI des projets du wiki.
