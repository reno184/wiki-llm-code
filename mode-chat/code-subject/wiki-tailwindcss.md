# Tailwind CSS — couleurs dynamiques du backend : safelist

## Problème

Le backend renvoie une couleur (ex. thème d'un objet). Vue.js interpole la valeur dans une classe dynamique :

```html
<div :class="'bg-' + item.color">…</div>
```

TailwindCSS **ne build pas** cette classe : le compilateur scanne les sources à la recherche de classes **littérales**. Une classe construite par interpolation au runtime est hors contexte → la couleur n'existe pas dans le CSS généré → rien ne s'affiche.

## Solution : safelist sans toucher à tailwind.config.js

Créer un petit fichier de safelist (ex. `src/assets/safelist.html`) contenant les classes littérales. Il est lu systématiquement par le compilateur Tailwind, qui les inclut alors dans le build.

```html
<!-- src/assets/safelist.html -->
<div class="
  shadow-primary border-primary bg-primary-content text-primary
  shadow-secondary border-secondary bg-secondary-content text-secondary
  shadow-accent border-accent bg-accent-content text-accent
  shadow-info border-info bg-info-content text-info
  shadow-success border-success bg-success-content text-success
  shadow-warning border-warning bg-warning-content text-warning
  shadow-error border-error bg-error-content text-error
"></div>
```

Variante : un simple commentaire dans le CSS/JS du projet listant les classes littérales fonctionne aussi (même principe : le scanner doit voir la chaîne complète en dur).

## Checklist « ne rien oublier »

- La classe complète (`bg-primary`, pas juste `primary`) doit apparaître **en littéral** quelque part scanné par Tailwind.
- Toute nouvelle couleur/nuance ajoutée côté backend doit être ajoutée à la safelist, sinon même bug silencieux.
- Alternative config : `safelist: []` dans `tailwind.config.js` — même effet, mais via le fichier safelist on garde le config intact.

## Liens croisés

- [wiki-vue-props-reactivity.md](wiki-vue-props-reactivity.md) — réactivité Vue / interpolation dynamique
- [wiki-variable-env.md](wiki-variable-env.md) — valeurs injectées au build vs runtime côté Vite (même famille de pièges : ce qui est visible à la compilation ≠ ce qui existe au runtime)
