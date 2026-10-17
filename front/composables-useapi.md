# Vue 3 — composable `useAPI` (fetch wrapper : loading, error, JSON)

Stack: Vue 3 (Composition API), fetch.
Usage: composable front qui encapsule les appels API — état `loading`/`error` réactif, parsing JSON, extraction du message d'erreur du backend. Un seul point d'entrée pour `fetch`, les composants ne font plus de fetch brut. Cohérent avec le errorHandler back ([../backend/express-error-handling.md](../backend/express-error-handling.md)) : 400 = JSON `{ code, message }` exploitable en popover, 500 dev = texte brut (stack).

## Le composable — `composables/useAPI.js`

```js
import { ref } from 'vue'

export const useAPI = () => {
    const loading = ref(false)
    const error = ref(null)

    const request = async (url, options = {}) => {
        loading.value = true
        error.value = null

        try {
            const rep = await fetch(url, options)

            if (!rep.ok) {
                const contentType = rep.headers.get('content-type') || ''

                if (contentType.includes('application/json')) {
                    const data = await rep.json()
                    throw new Error(data.message)
                }

                const text = await rep.text()
                throw new Error(text)
            }

            return await rep.json()
        } catch (err) {
            error.value = err.message
            throw err
        } finally {
            loading.value = false
        }
    }

    return { loading, error, request }
}
```

## Usage dans un composant

```vue
<script setup>
import { ref } from 'vue'
import { useAPI } from '@/composables/useAPI'

const { loading, error, request } = useAPI()
const materials = ref([])

const fetchMaterials = async () => {
    try {
        const data = await request('/api/materials')
        materials.value = data
    } catch (err) {
        // error.value est déjà peuplé par le composable — affichage popover/toast ici
    }
}
</script>

<template>
    <div v-if="loading">Chargement...</div>
    <div v-else-if="error" class="alert alert-error">{{ error }}</div>
    <ul v-else>...</ul>
</template>
```

## Upload multipart — le composable s'utilise tel quel

Les options de fetch passent directement — pour un upload, on lui donne le `FormData` de la fiche [upload-file.md](upload-file.md) :

```js
const { loading, error, request } = useAPI()

const submitUpload = async () => {
    try {
        const result = await request('/api/upload', {
            method: 'POST',
            body: bodyData.value
        })
        console.log('Uploadé :', result.url)
    } catch (err) {
        // popover "Fichier trop volumineux" si 400 du back (politique d'erreurs)
    }
}
```

Jamais de `Content-Type` manuel sur le body FormData (voir [upload-file.md](upload-file.md)).

## Notes

- **`useAPI()` s'appelle dans le `setup` du composant** (ou dans la fonction qui fait l'appel), pas au niveau module : chaque instance a son propre `loading`/`error`. Un singleton module exporterait un état partagé entre tous les composants — les chargements s'écraseraient mutuellement.
- **Le parsing d'erreur gère les deux formats du backend** : JSON pour les 400 (`{ code, message }` → popover avec le message métier, ex. "Fichier trop volumineux"), texte brut pour les 500 dev (le errorHandler renvoie la stack formatée en string — utile en dev pour voir l'erreur dans le toast).
- **Le composable re-throw** : `error.value` sert l'affichage réactif, le `throw` laisse le composant décider d'une action complémentaire (redirection, reset de formulaire). Le `catch` à l'appel est donc optionnel si on affiche juste `error`.
- **`finally` pour `loading`** : même en erreur, le spinner s'arrête.
- Pour les routes protégées, passer les headers dans les options : `headers: { Authorization: 'Bearer ...' }` (token via composable auth ou env).
- Point d'entrée unique : si un jour il faut un timeout, un retry, ou un base URL, on ne modifie QUE `useAPI` — aucun composant ne change.

## État

**vivant** — composable standard des fronts Vue 3 du wiki, aligné avec la politique d'erreurs back (400 JSON / 500 texte) et le trio upload.
