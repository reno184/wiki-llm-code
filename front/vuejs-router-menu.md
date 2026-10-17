# Routing Vue 3 + menu Tailwind/DaisyUI — navigation par pages

Stack: Vue 3, vue-router 4, TailwindCSS + DaisyUI (composant `menu`), Font Awesome.
Usage: navigation entre pages/vues d'un layout (`router-link` / `router-view`), menu horizontal actif, transition slide-fade entre pages.

## Architecture

- Un **layout parent** (`UserLayout`) contient le menu + le `<router-view>`. Les pages sont ses **enfants** (children) dans le router.
- Le menu est **piloté par un tableau de config** `menus` : une seule source de vérité pour path, label, libellé court et icône.
- `linkActiveClass` du router branche la classe d'état actif de DaisyUI sur chaque `<router-link>`.

## Code

### 1. Config du menu — `UserLayout.vue` (script)

```js
const menus = [
  { path: '/reviewMaterial', label: 'Review material', short: 'Review', icon: 'fa-arrows-h' },
  { path: '/searchMaterial', label: 'Search material', short: 'Search', icon: 'fa-search' },
  { path: '/createMaterial', label: 'Create material', short: 'Create', icon: 'fa-plus-circle' }
];
```

- `label` : affiché en desktop, `short` : affiché en mobile (libellé court pour les petits écrans), `icon` : classe Font Awesome (préfixe `fas` séparé dans le template).

### 2. Menu DaisyUI — template

```html
<ul class="menu menu-horizontal rounded-box">
  <li v-for="menu in menus" :key="menu.path">
    <router-link :to="menu.path">
      <i class="fas" :class="menu.icon"></i>
      <span class="inline md:hidden">{{ menu.short }}</span>
      <span class="hidden md:inline">{{ menu.label }}</span>
    </router-link>
  </li>
</ul>
```

- `menu menu-horizontal rounded-box` : composant menu horizontal DaisyUI.
- `<span class="inline md:hidden">` / `hidden md:inline` : bascule mobile/desktop en pur Tailwind, sans JS.

### 3. Router — `router/index.js`

```js
import { createRouter, createWebHistory } from 'vue-router';
import UserLayout from '@/layouts/UserLayout.vue';
import ReviewMaterialPage from '@/pages/ReviewMaterialPage.vue';
import SearchMaterialPage from '@/pages/SearchMaterialPage.vue';
import CreateMaterialPage from '@/pages/CreateMaterialPage.vue';

const router = createRouter({
  history: createWebHistory(),
  linkActiveClass: 'menu-active',
  routes: [
    {
      path: '/',
      component: UserLayout,
      children: [
        { path: '/reviewMaterial', name: 'reviewMaterial', component: ReviewMaterialPage },
        { path: '/searchMaterial', component: SearchMaterialPage },
        { path: '/createMaterial', component: CreateMaterialPage }
      ]
    }
  ]
});
```

**`linkActiveClass: 'menu-active'` — point clé avec Tailwind/DaisyUI.**
Par défaut vue-router applique la classe `router-link-active` au lien de la route courante. Avec le composant `menu` de DaisyUI, l'état actif est géré par la classe CSS `menu-active` : on surcharge donc `linkActiveClass` pour que vue-router pose directement `menu-active` sur le `<router-link>` de la page courante. DaisyUI style alors l'onglet actif (fond/surlignage) **sans aucune classe utilitaire ni :class à écrire**. Les enfants d'une route parente héritent aussi de la classe active (linkActiveClass s'applique au lien actif le plus proche ; utiliser `linkExactActiveClass` si besoin d'une correspondance exacte).

### 4. `<router-view>` + transition — `UserLayout.vue`

```html
<router-view v-slot="{ Component, route }">
  <transition name="slide-fade" mode="out-in">
    <component :is="Component" :key="route.path" />
  </transition>
</router-view>
```

- `v-slot` expose `Component` (la page à rendre) et `route` (la route courante).
- `<component :is="Component">` : rendu dynamique de la page.
- `:key="route.path"` : **indispensable** — force Vue à recréer le composant quand on navigue entre deux pages (sinon même composant = pas de transition, réutilisation d'instance).
- `mode="out-in"` : l'ancienne page sort (`leave`) avant que la nouvelle n'entre (`enter`) — évite le saut de layout, les deux pages ne sont jamais empilées.

### 5. Transition slide-fade — `<style scoped>`

```css
/* Animation Slide & Fade façon Apple / iCloud */
.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.5s cubic-bezier(0.25, 0.8, 0.25, 1);
}

.slide-fade-enter-from {
  opacity: 0;
  transform: translateX(15px);
}

.slide-fade-leave-to {
  opacity: 0;
  transform: translateX(-15px);
}
```

- La nouvelle page arrive depuis la droite (`translateX(15px)` → 0), l'ancienne sort vers la gauche (0 → `-15px`), avec fondu. Le `cubic-bezier(0.25, 0.8, 0.25, 1)` donne l'easing doux "Apple".

## Notes

### ⚠️ Piège n°1 : un seul élément racine par page

**Les composants enfants (pages) doivent avoir un unique élément racine** pour que la transition fonctionne. `<transition>` ne sait animer qu'un seul élément : si une page a plusieurs racines (fragments), Vue ne peut pas garantir l'ordre out/in et la transition ne se déclenche pas (warning console). Toujours envelopper le template de chaque page dans un seul `<div>` (ou `<section>`).

### ⚠️ Piège n°2 : oublier `:key="route.path"`

Sans `:key`, si deux routes partagent le même composant (ou si Vue réutilise l'instance), aucun remount → pas de transition. La key sur `route.path` force le cycle enter/leave à chaque navigation.

### Chaîne complète

`router-link` → navigation → `route.path` change → key change → `<transition>` lance `slide-fade-leave` sur l'ancienne page (`out`) puis `slide-fade-enter` sur la nouvelle (`in`) ; en parallèle vue-router pose `menu-active` sur le lien courant, DaisyUI style l'onglet actif.

- Les `path` des children peuvent être absolus (`/reviewMaterial`) même imbriqués : pas besoin de préfixe relatif.
- Mettre un `name` sur les routes qu'on vise avec `router.push({ name: ... })` ; `router-link :to="menu.path"` suffit pour le menu.

## État

**vivant** — implémentation réelle (navigation material, layout DaisyUI).
