# Validation de formulaire — HTML natif + VueUse (Vue 3)

## Le CSS — obligatoire dans main.css

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

## Pattern — composant

```html
<form @submit.prevent="handleSubmit">
    <input type="text" name="text" required minlength="2" class="input input-bordered">
    <button type="submit" class="btn btn-primary">Envoyer</button>
</form>
```

```js
const bodyData = ref(null)

const { isFetching: pending, execute } = useApi('/api/contact', { immediate: false }).post(bodyData).json()

const handleSubmit = async (e) => {
    const isValid = e.target.checkValidity()
    const formData = new FormData(e.target)

    if (isValid) {
        bodyData.value = {
            text: formData.get('text'),
        }

        await execute()
        e.target.reset()
    }
}
```

## État

**vivant** — mécanique de formulaire standard des projets Vue 3, validée en réel. Multi-étapes (`button.next`) : le CSS le couvre déjà, à consolider le jour venu si besoin.
