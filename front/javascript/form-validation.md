# Validation de formulaire — HTML natif + VueUse (Vue 3)

Stack: Vue 3 Composition API, @vueuse/core, API Constraint Validation du navigateur (pas de lib de validation).
Usage: mécanique de formulaire Vue 3 — champs réactifs, soumission, reset — validé par les **contraintes HTML natives** (`required`, `type`, `minlength`, `pattern`). Le navigateur bloque la soumission et affiche ses bulles, le CSS rend le bouton inactif tant que le formulaire est invalide. Fiche du profil Développeur JavaScript ; la soumission passe par [vuejs-useapi.md](../vuejs-useapi.md) (400 serveur = snackbar, 5xx = page 500, zéro try/catch).

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
- **Pas de v-model sur les champs du formulaire validé nativement** : les valeurs sont lues via `new FormData(e.target)` au submit — le HTML attributs + FormData remplace l'état réactif champ par champ. `v-model` reste pour les cas hors validation native (ex. champ de recherche instantanée).
- **`e.target.reset()` après l'envoi** : le formulaire est vidé après un submit réussi — remise à zéro native, repart au CSS `form:invalid` (bouton à nouveau inactif).

## Pattern — composant

```html
<form @submit.prevent="handleSubmit">
    <input type="text" name="text" required minlength="2" class="input input-bordered">
    <button type="submit" class="btn btn-primary">Envoyer</button>
</form>
```

```js
const { isFetching: pending, execute } = useApi('/api/contact').post(body).json()

const handleSubmit = async (e) => {
    const isValid = e.target.checkValidity()
    const formData = new FormData(e.target)

    if (isValid) {
        const body = {
            text: formData.get('text'),
        }

        await execute()
        e.target.reset()
    }
}
```

- `e.target` = le `<form>` : `checkValidity()` et `new FormData(e.target)` travaillent directement sur le DOM — zéro état réactif à maintenir pour les champs.
- `checkValidity()` dans le handler est une **garde défensive** : sans `novalidate`, le navigateur bloque l'invalidité avant l'événement, donc `isValid` est en pratique toujours `true` — mais la garde rend le handler autonome et lisible (le `if` fait foi : rien ne part si invalide).
- `formData.get(name)` : un champ par clé `name` — le body est un objet JS explicite (pas le FormData brut) pour un POST JSON.
- `await execute()` : on attend la fin de la requête avant le `reset()` — le FormData est déjà lu avant l'appel, le reset ne peut pas casser la requête.
- `e.target.reset()` : vide les champs, le CSS `form:invalid` reprend (bouton inactif), l'utilisateur repart d'un formulaire propre.
- Le CSS de main.css rend le bouton inactif tant que `form:invalid` ; `pending` reste utilisable pour un spinner/loader séparé si besoin.

## Cas particulier : input file (pont avec upload-file.md)

- Pas de v-model sur un input file — pas non plus besoin du pattern `handleChange` : `new FormData(e.target)` au submit embarque directement le `File` (voir [upload-file.md](../upload-file.md)).
- `required` sur l'input file participe à `form:invalid` : tant qu'aucun fichier n'est sélectionné, le bouton reste inactif (CSS) et la soumission serait de toute façon bloquée nativement.
- Dès qu'il y a un fichier, le FormData embarque le `File` — le `beforeFetch` du wrapper retire le Content-Type et le navigateur pose le boundary multipart ([vuejs-useapi.md](../vuejs-useapi.md)).

## Anti-patterns

- VeeValidate/Zod pour un formulaire couvert par les attributs natifs.
- `novalidate` : réimplémente à la main ce que le navigateur fait déjà.
- `:disabled` ou `disabled` JS sur le bouton submit — le CSS `form:invalid` de main.css est le seul mécanisme.
- Bouton secondaire sans `type="button"` dans le form (tout `<button>` y est `submit` implicite → touché par le CSS `form:invalid`).
- Oublier le bloc CSS dans main.css : sans lui, le bouton paraît actif alors que la soumission est bloquée par le navigateur.
- v-model champ par champ sur un formulaire validé nativement (double source de vérité avec FormData) — sauf besoin réactif explicite.
- `reset()` avant `await execute()` : le FormData est déjà lu, mais l'ordre échoue silencieusement sur une requête rejetée/annulée.
- Messages custom stockés en dur : `validationMessage` est déjà localisé par le navigateur.

## Notes

- Le serveur revalide toujours (le front est contournable) — ses 400 JSON arrivent en snackbar via le wrapper.
- `form:invalid` s'applique aussi aux formulaires multi-étapes si chaque étape est un `<form>` séparé ; `button.next` est couvert par le CSS (blocage de l'étape tant que ses champs sont invalides).
- Le clic sur un bouton visuellement inactif ne passe pas (`pointer-events: none`) : pas de gestionnaire à écrire pour ce cas.
- Liens : [vuejs-useapi.md](../vuejs-useapi.md) (soumission + erreurs serveur), [upload-file.md](../upload-file.md) (input file + FormData), [daisyui-components.md](../daisyui-components.md) (input, label, btn).

## État

**en construction** — pattern posé avec le mécanisme de formulaire en cours ; consolidation avec le code réel validé (notamment multi-étapes et le rendu du bloc CSS avec Tailwind).
