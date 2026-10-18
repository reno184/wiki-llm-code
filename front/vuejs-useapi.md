# Composable useApi — wrapper createFetch de VueUse (Vue 3)

Stack: Vue 3 Composition API, @vueuse/core (`createFetch`), fetch natif (pas d'axios).
Usage: `useApi` n'est **pas** un composable maison — c'est le decorator de la librairie VueUse : `createFetch` fabrique un wrapper de fetch préconfiguré (baseUrl, headers, gestion d'erreurs globale). Tous les composants appellent `useApi(url)` au lieu de `fetch`. Branchement direct sur la politique d'erreurs back ([../backend/express-error-handling.md](../backend/express-error-handling.md)) : 400 JSON = snackbar, 500 texte = page d'erreur.

```bash
npm install @vueuse/core
```

## Le wrapper — `composables/useApi.js`

```js
import { createFetch } from '@vueuse/core'

function dispatchHttpError(error) {
    if (error.name !== 'AbortError') {
        const fullStack = [
            '=== APPELANT (REFERER) ===',
            error.stack,
            error.serverStack || ''
        ].join('\n')

        const errorEvent = new CustomEvent('redirect-error-500-page', { detail: { stack: fullStack } })

        window.dispatchEvent(errorEvent)

        throw error
    }
}

export const useApi = createFetch({
    baseUrl: 'http://localhost:3000',
    options: {
        async beforeFetch({ options }) {
            if (options.body instanceof FormData) {
                delete options.headers['Content-Type']
            }
            return { options }
        },
        async onFetchError(ctx) {
            const response = ctx.response

            if (response) {
                const status = response.status

                if (status === 400) {
                    const data = await response.clone().json()
                    window.dispatchEvent(new CustomEvent('show-snackbar', {
                        detail: { message: data.message || 'Erreur dans la requête', type: 'error' }
                    }))
                } else if (status >= 500) {
                    try {
                        const serverStack = await response.clone().text()
                        const error = new Error(`Server Error (${status})`)
                        error.serverStack = serverStack
                        dispatchHttpError(error)
                    } catch (e) {
                        dispatchHttpError(e)
                    }
                }
            }

            return ctx
        }
    }
})
```

## Utilisation dans un composant

```js
const { data, error, isFetching, execute } = useApi('/api/studyMaterial/list').get().json()
```

- `useApi(url)` retourne un objet réactif VueUse : `data`, `error`, `isFetching`, `execute` — pas besoin de réécrire des refs loading/error.
- `.get().json()`, `.post(body)`, `.put(body)` : chaînage VueUse.

## POST JSON / POST FormData (upload)

```js
await useApi('/api/material/newNovel').post(bodyData.value).json()
```

- `beforeFetch` retire le `Content-Type` si le body est un FormData : le navigateur pose lui-même le boundary multipart (voir [upload-file.md](upload-file.md)). C'est le piège documenté côté front, il est neutralisé **une fois pour toutes dans le wrapper**.

## Architecture des erreurs — le cœur du pattern

La gestion d'erreur est **globale, dans le wrapper** — les composants n'ont pas de try/catch à écrire. Le format de réponse fait le tri :

| Statut back | Format (errorHandler) | Action front |
|---|---|---|
| 400 | JSON `{ code, message }` | `show-snackbar` CustomEvent → message métier en snackbar (ex. "Fichier trop volumineux") |
| 500 dev | texte (stack formatée) | `redirect-error-500-page` CustomEvent avec la stack serveur concaténée à la stack appelante → page d'erreur |
| 500 prod | `Internal Server Error` | même événement — page d'erreur générique |
| AbortError | — | ignoré volontairement (annulation volontaire de requête, VueUse/timeout) |

- **`dispatchHttpError`** : concatène la stack front (appelant) et la stack serveur (`error.serverStack` — le texte du errorHandler dev) dans un seul bloc, puis émet `redirect-error-500-page`. Le composant racine écoute ce CustomEvent et redirige vers la page 500 en affichant la stack en dev.
- **`response.clone()`** : obligatoire avant `.json()`/`.text()` — le body ne se lit qu'une fois, le clone laisse le ctx intact pour VueUse.
- Les composants consomment `error` (réactif) pour les états locaux, mais **l'affichage des erreurs passe par les CustomEvents** : snackbar (400) et page 500 (5xx) sont gérés une seule fois au niveau App, pas dupliqués par composant.

## Notes

- **Ne pas réécrire un composable fetch maison** tant que VueUse fournit `createFetch` : la librairie gère l'abort, les refs réactives, le chaînage — le wrapper ne fait que configurer (baseUrl, erreurs, FormData).
- `baseUrl` : à passer par env en prod (ex. `import.meta.env.VITE_API_URL`), pas en dur.
- **Le `delete` du Content-Type dans `beforeFetch` est le piège n°1 du wrapper** : `createFetch` pose `Content-Type: application/json` par défaut sur chaque requête — un FormData doit au contraire laisser le navigateur générer son boundary multipart. Sans ce delete, l'upload casse (le backend rejette ou parse mal le multipart). Ce code est **l'exception Règle 0** : mérite un commentaire à lui tout seul, et les agents doivent poser le commentaire figé au point d'appel côté composant (voir [express-strategy.md](express-strategy.md), garde 2).
- Un nouvel événement d'erreur = un nouveau `CustomEvent` dans le wrapper + un écouteur au niveau App : jamais de gestion d'erreur dans les composants.
- Le `AbortError` est exclu : VueUse annule les requêtes auUnmount par défaut — ce n'est pas une vraie erreur.
- Liens : [vuejs-router-menu.md](vuejs-router-menu.md) pour la page 500 dans le router, [daisyui-components.md](daisyui-components.md) pour la snackbar/alert.

## État

**vivant** — wrapper createFetch VueUse standard des projets Vue 3, aligné avec le errorHandler back (400 JSON / 500 texte) et le trio upload.
