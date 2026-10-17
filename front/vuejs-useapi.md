# Composable useAPI — appels fetch réutilisables (Vue 3)

Stack: Vue 3 Composition API, fetch natif (pas d'axios).
Usage: pattern de composable pour tous les appels API d'un projet Vue — encapsuler fetch dans un `useAPI` réutilisable, avec gestion des états `loading`/`error` et branchement sur la popover d'erreur du errorHandler (fiche [../backend/express-error-handling.md](../backend/express-error-handling.md) : 400 = erreur probable affichée, 500 = bug serveur).

## Pourquoi un composable useAPI

Sans composable, chaque appel réécrit fetch + try/catch + états loading/error. Avec un `useAPI`, la logique réseau est écrite une fois et les composants consomment `useAPI()`.

Rappel : un composable = une fonction `use...()` qui expose des refs. Ne pas transformer ça en couche service/api complexe — le composable reste simple, de haut en bas, pas de commentaires (Règle 0 du wiki).

## Pattern minimal — `composables/useAPI.js`

```js
import { ref } from 'vue'

export function useAPI() {
    const data = ref(null)
    const loading = ref(false)
    const error = ref(null)

    const request = async (url, options = {}) => {
        loading.value = true
        error.value = null

        try {
            const rep = await fetch(url, options)

            if (!rep.ok) {
                const body = await rep.json().catch(() => null)
                error.value = body?.message ?? `Erreur ${rep.status}`
                return null
            }

            data.value = await rep.json()
            return data.value
        } catch (err) {
            error.value = 'Erreur réseau'
            return null
        } finally {
            loading.value = false
        }
    }

    return { data, loading, error, request }
}
```

## Utilisation dans un composant

```vue
<script setup>
import { useAPI } from '@/composables/useAPI'

const { data, loading, error, request } = useAPI()

const loadMaterials = () => request('/api/studyMaterial/list')
</script>

<template>
    <div v-if="loading">Chargement...</div>
    <div v-else-if="error" class="alert alert-error">{{ error }}</div>
    <div v-else>...</div>
</template>
```

- `alert alert-error` : composant DaisyUI (voir [daisyui-components.md](daisyui-components.md)).
- Les états (`loading`, `error`) sont des refs réactives, directement utilisables dans le template.

## Appels spécifiques

### POST JSON

```js
await request('/api/material', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ title, author })
})
```

### POST FormData (upload)

```js
await request('/api/studyMaterial/newNovel', {
    method: 'POST',
    body: formData
})
```

⚠️ **Pas de header Content-Type avec FormData** (voir [upload-file.md](upload-file.md)) — le navigateur pose le boundary multipart. Le composable n'ajoute aucun header par défaut, c'est à l'appelant de définir les options.

### GET avec query params

```js
const params = new URLSearchParams({ search, page })
await request(`/api/material?${params}`)
```

## Branchement sur la politique d'erreurs du back

Le errorHandler back renvoie `{ code, message }` en JSON sur les 400 (voir [../backend/express-error-handling.md](../backend/express-error-handling.md)) :

- **400** → `error.value = body.message` — une erreur probable, on affiche le message serveur directement dans la popover/alert (ex. "Fichier trop volumineux").
- **500** → le back renvoie `Internal Server Error` en texte brut (pas de JSON en prod) — `body?.message` est `undefined`, on affiche le fallback générique. C'est un bug serveur, pas un message utilisateur.
- **Réseau (serveur down, offline)** → catch : `error.value = 'Erreur réseau'`.

Le front n'a donc **pas besoin de connaître le code HTTP** : le format de réponse fait le tri (JSON parsable = erreur exploitable, texte brut = erreur générique).

## Notes

- **Un seul useAPI par projet** — pas un composable par endpoint. Les options (method, headers, body) suffisent à couvrir tous les cas.
- Le composable n'ajoute **aucun header par défaut** (pas de Content-Type automatique, pas d'Authorization) — chaque appel définit ses options explicitement.
- Si auth JWT : ajouter le token dans les options à l'appelant (`headers: { Authorization: \`Bearer ${token}\` }`) ou via un intercepteur maison dans `request` — selon les besoins du projet, pas de sur-ingénierie par défaut (style proto, voir [../patterns/express-strategy.md](../patterns/express-strategy.md)).
- `await rep.json().catch(() => null)` : le back en prod ne renvoie pas toujours du JSON (500 = texte brut), le catch protège le parse.
- Pas d'axios : fetch natif suffit, une dépendance de moins.
- Éviter la dérive "usine à gaz" : pas de retry, pas de cache, pas de type generique — le composable reste ~30 lignes.

## État

**vivant** — pattern front standard pour tous les appels API, aligné avec la politique d'erreurs 400/500 du errorHandler back.
