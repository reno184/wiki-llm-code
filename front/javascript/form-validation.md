# Validation de formulaire — HTML natif + VueUse (Vue 3)

Stack: Vue 3 Composition API, @vueuse/core, API Constraint Validation du navigateur (pas de lib de validation).
Usage: mécanique de formulaire Vue 3 — champs réactifs, soumission, erreurs — validé par les **attributs HTML natifs** (`required`, `type`, `minlength`, `pattern`) pilotés manuellement (`checkValidity` / `validationMessage`) pour brancher le styling DaisyUI. Fiche du profil Développeur JavaScript ; la soumission passe par [vuejs-useapi.md](../vuejs-useapi.md) (400 serveur = snackbar, 5xx = page 500, zéro try/catch).

## Décisions

- **Validation déclarative HTML native, pas de lib** (VeeValidate, Zod front : rejetés). Le navigateur valide via les attributs, les messages = `el.validationMessage` (localisés par l'OS, zéro stockage). Une lib ne se justifie que sur une validation croisée entre champs réelle (ex. plage de dates) → nouvelle fiche le jour venu.
- **`novalidate` + déclenchement manuel** : les contraintes restent natives mais c'est le composant qui décide quand valider — impossible de styler `input-error` DaisyUI avec la bulle native du navigateur.
- **Erreur posée à la soumission, retirée à la correction** : jamais de validation à chaque frappe avant la première soumission ; à la correction d'un champ, sa seule erreur est réévaluée (`watchDebounced` VueUse).
- **Bouton jamais disabled sur l'invalidité** : le clic sur un formulaire invalide est le moyen le plus rapide de montrer l'erreur (`reportValidity` focus le premier champ en erreur). `:disabled` uniquement sur `pending`.

## Pattern — composant

```html
<form ref="formRef" novalidate class="flex flex-col gap-4" @submit.prevent="onSubmit">
    <div class="form-control">
        <input v-model="fields.email" name="email" type="email" required class="input input-bordered" :class="{ 'input-error': errors.email }">
        <label class="label"><span class="label-text-alt text-error">{{ errors.email }}</span></label>
    </div>
    <div class="form-control">
        <input v-model="fields.name" name="name" required minlength="2" class="input input-bordered" :class="{ 'input-error': errors.name }">
        <label class="label"><span class="label-text-alt text-error">{{ errors.name }}</span></label>
    </div>
    <button type="submit" class="btn btn-primary" :disabled="pending">Envoyer</button>
</form>
```

```js
import { reactive, ref } from 'vue'
import { watchDebounced } from '@vueuse/core'

const formRef = ref(null)
const fields = reactive({ email: '', name: '' })
const errors = reactive({ email: '', name: '' })

watchDebounced(fields, () => {
    for (const el of formRef.value?.elements ?? []) {
        if (el.name && errors[el.name]) {
            errors[el.name] = el.checkValidity() ? '' : el.validationMessage
        }
    }
}, { debounce: 300 })

function onSubmit() {
    const form = formRef.value

    if (!form.checkValidity()) {
        form.reportValidity()
        for (const el of form.elements) {
            if (el.name) errors[el.name] = el.checkValidity() ? '' : el.validationMessage
        }
        return
    }

    for (const key of Object.keys(errors)) errors[key] = ''
    submit()
}

const { isFetching: pending, execute: submit } = useApi('/api/contact').post(fields).json()
```

- `watchDebounced` ne réévalue que les champs **déjà en erreur** (`if (errors[el.name])`) — pas de validation précoce sur les champs sains.
- `reportValidity()` en plus de `checkValidity()` : affiche aussi la bulle native sur le premier champ en erreur (double feedback, bulle + styling DaisyUI).

## Cas particulier : input file (pont avec upload-file.md)

- Pas de v-model sur un input file — `File` gardé via `@change`, `FormData` construite à la correction/au submit (voir [upload-file.md](../upload-file.md)).
- Dans un formulaire validé, l'input file porte `required` comme les autres et participe à `checkValidity()` — "aucun fichier sélectionné" est couvert par le message natif.
- Dès qu'il y a un fichier, le body est un `FormData` (pas de JSON) — le `beforeFetch` du wrapper retire le Content-Type ([vuejs-useapi.md](../vuejs-useapi.md)).

## Anti-patterns

- VeeValidate/Zod pour un formulaire couvert par les attributs natifs.
- Valider à chaque frappe avant la première soumission.
- `disabled` sur le bouton tant que le formulaire est invalide (bloque la découverte des erreurs).
- Bouton secondaire sans `type="button"` dans le form (tout `<button>` y est `submit` implicite).
- Messages custom stockés en dur : `validationMessage` est déjà localisé par le navigateur.
- v-model sur un input file.

## Notes

- `form.elements` : HTMLFormControlsCollection itérable — `el.name` filtre les champs sans name (dont les boutons).
- `validationMessage` = chaîne vide si le champ est valide, le ternaire n'est qu'une garde de lisibilité.
- Message custom si besoin : `el.setCustomValidity('...')` AVANT `checkValidity()` — cas séparé, jamais par défaut.
- Le serveur revalide toujours (le front est contournable) — ses 400 JSON arrivent en snackbar via le wrapper.
- `watchDebounced` 300 ms : compromis validé à l'usage, pas une règle absolue.
- Liens : [vuejs-useapi.md](../vuejs-useapi.md) (soumission + erreurs serveur), [upload-file.md](../upload-file.md) (input file + FormData), [daisyui-components.md](../daisyui-components.md) (input, label, btn).

## État

**en construction** — pattern posé avec le mécanisme de formulaire en cours ; consolidation avec le code réel validé (notamment debounce de réévaluation et double reportValidity).
