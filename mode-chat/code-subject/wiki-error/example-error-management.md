# Gestion d'erreur personnalisÃ©e â du micro-service Python Ã  la snackbar front

> Exemple complet du trajet d'une erreur mÃ©tier de bout en bout : `ValueError` levÃ©e dans un micro-service Python â HTTP 400 JSON â middleware d'erreurs Express â `onFetchError` du composable `useApi` (VueUse) â snackbar visible par l'utilisateur.

Cas concret : prÃ©venir l'utilisateur que le rectangle de clip posÃ© sur un PDF ne permet de trouver aucun mot.

Fiches liÃ©es :
- [express-error-handling.md](../express-error-handling.md) â le pattern centralisÃ© Express (`AppError` + `errorHandler`), dont ce middleware est un extrait
- [anonymous-rectangle-clip.md](../../use-case/anonymous-rectangle-clip.md) â la suppression rÃ©elle du texte par rectangle de rÃ©daction (vs simple masque)

## 1. Micro-service Python â lever l'erreur mÃ©tier (`searchWordService.py`)

```python
def words_in_rect(doc: pymupdf.Document, req_body: Dict) -> List[List[Dict[str, Any]]]:
    # existing code...

    text_under_clip = current_page.get_text("text", clip=rect_clip).strip()

    if not text_under_clip:
        raise ValueError("Aucun texte trouvÃ© dans la zone sÃ©lectionnÃ©e")

    return words2_in_clip(doc, text_under_clip, True)
```

## 2. Micro-service Python â traduire en HTTP 400 (`searchWordController.py`)

```python
def handle_searchWord_byRect():
    try:
        # existing code...

        results = words_in_rect(doc, rectangleBody)

        return jsonify(results), 200

    except ValueError as e:
        return jsonify({"message": str(e)}), 400
    except Exception:
        stack = traceback.format_exc()
        return Response(stack, status=500, mimetype='text/plain')
```

## 3. Backend Node â propager au middleware (`searchWordController.js`)

```js
export async function byRectMinioController(req, res, next) {
    try {
        // existing code...

        const response = await callPython('/searchWord/byRect', formData)

        const data = await response.json()

        res.json(data)
    } catch (err) {
        next(err)
    }
}
```

## 4. Backend Node â middleware d'erreurs (`errorHandler`)

```js
export function errorHandler(err, req, res, _next) {
    if (err.status === 400 || err.status === '400') {
        res.status(400).json({ code: 400, message: err.message })
    } else {
        // existing code...
    }
}
```

## 5. Front â intercepter dans `useApi` et Ã©mettre l'Ã©vÃ©nement snackbar

```js
export const useApi = createFetch({
    baseUrl: 'http://localhost:3000',
    options: {
        // existing code...
        async onFetchError(ctx) {
            const response = ctx.response

            if (response) {
                const status = response.status

                if (status === 400) {
                    const data = await response.clone().json()
                    window.dispatchEvent(new CustomEvent('show-snackbar', { detail: { message: data.message, type: 'error' } }))
                } else if (status >= 500) {
                    // existing code...
                }
            }

            return ctx
        }
    }
})
```

## 6. Front â Ã©couter l'Ã©vÃ©nement dans `App.vue`

```html
<template>
  <router-view/>
  <MySnackbar ref="snackbarRef" baseIconError="fa fa-exclamation-triangle"></MySnackbar>
</template>

<script setup>
const snackbarRef = ref(null)

const executeSnackbar = (payload) => {
  const { message, type } = payload.detail
  snackbarRef.value.trigger(message, type)
}

onMounted(() => {
  window.addEventListener('show-snackbar', executeSnackbar)
})

onUnmounted(() => {
  window.removeEventListener('show-snackbar', executeSnackbar)
})
</script>
```
