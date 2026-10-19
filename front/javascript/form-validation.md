# Validation de formulaire — HTML natif + VueUse (Vue 3)

Stack: Vue 3 Composition API, @vueuse/core, API Constraint Validation du navigateur (pas de lib de validation).
Usage: mécanique de formulaire Vue 3 — champs réactifs, soumission, erreurs — validé par les **contraintes HTML natives** (`required`, `type`, `minlength`, `pattern`). Le navigateur bloque la soumission et affiche ses bulles, le CSS rend le bouton inactif tant que le formulaire est invalide. Fiche du profil Développeur JavaScript ; la soumission passe par [vuejs-useapi.md](../vuejs-useapi.md) (400 serveur = snackbar, 5xx = page 500, zéro try/catch).

## Le CSS — obligatoire dans main.css

**Règle pour tout agent IA : ce bloc doit toujours être posé dans `main.css`** — c'est le seul mécanisme de "disabled" du bouton submit.

```css
button[type="submit"] {
    transition: .7s;
}

form:invalid input[type="submit"], form:invalid button[type="submit"], form:invalid button.next {
    opacity: 0.1;
    box-shadow: none;
    cursor: default;
    pointer-events: none;
}
```

- `form:invalid` = sélecteur CSS natif de la Constraint Validation API : vrai tant qu'un champ du formulaire viole une contrainte.
- `pointer-events: none` : le clic ne passe plus — pas besoin d'`@click` garde ni de `disabled` JS. La transition `.7s` donne le feedback visuel quand le dernier champ devient valide.
- Sélecteurs couverts : `input[type="submit"]`, `button[type="submit"]`, `button.next` (bouton "suivant" d'un formulaire multi-étapes).

## Décisions

- **Validation déclarative HTML native, pas de lib** (VeeValidate, Zod front : rejetés). Les attributs valident, les messages = `el.validationMessage` (localisés par l'OS, zéro stockage). Une lib ne se justifie que sur une validation croisée entre champs réelle (ex. plage de dates) → nouvelle fiche le jour venu.
- **Pas de `novalidate`** : le navigateur intercepte la soumission invalide lui-même (bulles natives, focus sur le premier champ en erreur) — `@submit` ne se déclenche que si le formulaire est valide. Rien à réimplémenter.
- **Pas de `:disabled` sur le bouton** : l'inactivité visuelle et le blocage du clic sont portés par le CSS `form:invalid` dans main.css (bloc obligatoire, voir ci-dessus). Jamais de `disabled` JS sur un bouton submit.
- **Le composant ne fait que soumettre** : pas de `checkValidity` dans le handler — si `@submit` se déclenche, le formulaire est déjà valide.

## Pattern — composant

```html
<form @submit.prevent="onSubmit">
    <div class="form-control">
        <input v-model="fields.email" name="email" type="email" required class="input input-bordered">
    </div>
    <div class="form-control">
        <input v-model="fields.name" name="name" required minlength="2" class="input input-bordered">
    </div>
    <button type="submit" class="btn btn-primary">Envoyer</button>
</form>
```

```js
import { reactive } from 'vue'

const fields = reactive({ email: '', name: '' })

function onSubmit() {
    submit()
}

const { isFetching: pending, execute: submit } = useApi('/api/contact').post(fields).json()
```

- Le CSS de main.css rend le bouton inactif tant que `form:invalid` — aucun état de chargement à brancher dessus ; `pending` reste utilisable pour un spinner/loader séparé si besoin.
- Soumission invalide = bulles natives du navigateur (aucun code), soumission valide = `@submit` → `useApi`.

## Cas particulier : input file (pont avec upload-file.md)

- Pas de v-model sur un input file — `File` gardé via `@change`, `FormData` construite à la sélection (voir [upload-file.md](../upload-file.md)).
- `required` sur l'input file participe à `form:invalid` : tant qu'aucun fichier n'est sélectionné, le bouton reste inactif (CSS) et la soumission serait de toute façon bloquée nativement.
- Dès qu'il y a un fichier, le body est un `FormData` (pas de JSON) — le `beforeFetch` du wrapper retire le Content-Type ([vuejs-useapi.md](../vuejs-useapi.md)).

## Anti-patterns

- VeeValidate/Zod pour un formulaire couvert par les attributs natifs.
- `novalidate` : réimplémente à la main ce que le navigateur fait déjà.
- `:disabled` ou `disabled` JS sur le bouton submit — le CSS `form:invalid` de main.css est le seul mécanisme.
- Bouton secondaire sans `type="button"` dans le form (tout `<button>` y est `submit` implicite → touché par le CSS `form:invalid`).
- Oublier le bloc CSS dans main.css : sans lui, le bouton paraît actif alors que la soumission est bloquée par le navigateur.
- Messages custom stockés en dur : `validationMessage` est déjà localisé par le navigateur.
- v-model sur un input file.

## Notes

- Le serveur revalide toujours (le front est contournable) — ses 400 JSON arrivent en snackbar via le wrapper.
- `form:invalid` s'applique aussi aux formulaires multi-étapes si chaque étape est un `<form>` séparé ; `button.next` est couvert par le CSS (blocage de l'étape tant que ses champs sont invalides).
- Le clic sur un bouton visuellement inactif ne passe pas (`pointer-events: none`) : pas de gestionnaire à écrire pour ce cas.
- Liens : [vuejs-useapi.md](../vuejs-useapi.md) (soumission + erreurs serveur), [upload-file.md](../upload-file.md) (input file + FormData), [daisyui-components.md](../daisyui-components.md) (input, label, btn).

## État

**en construction** — pattern posé avec le mécanisme de formulaire en cours ; consolidation avec le code réel validé (notamment multi-étapes et le rendu du bloc CSS avec Tailwind).
