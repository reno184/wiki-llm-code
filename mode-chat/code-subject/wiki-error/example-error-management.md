Ceci est un exemple de la gestion d'une error personnalisé attrapé dans un micro-service méthod python jsuqu'à l'afficahge dans une snackbar pour prévenir un utilisateur

Le message permet de prévenir que le rectangle clip sur un pdf ne permet de trouver un mots.

Micro service : searchWordService.py
```python
def words_in_rect(doc: pymupdf.Document, req_body: Dict) -> List[List[Dict[str, Any]]]:
   
  # existing code...

    text_under_clip = current_page.get_text("text", clip=rect_clip).strip()

    if not text_under_clip:
        raise ValueError("Aucun texte trouvé dans la zone sélectionnée")

    return words2_in_clip(doc, text_under_clip, True)
```

Micro service : searchWordController.py
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

Backend : searchWordController.js

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

Backend: Error middleware

```js
export function errorHandler(err, req, res, _next) {
    if (err.status === 400 || err.status === '400') {
        res.status(400).json({code: 400, message: err.message})
    } else {
       // existing code
    }
};
```

frontend: composable useApi.js
```js
export const useApi = createFetch({
    baseUrl: 'http://localhost:3000',
    options: {
       // existing code...
        },
        async onFetchError(ctx) {
            const response = ctx.response

            if (response) {
                const status = response.status

                if (status === 400) {

                    const data = await response.clone().json()
                    window.dispatchEvent(new CustomEvent('show-snackbar', {detail: {message: data.message, type: 'error'}}))
                } else if (status >= 500) {
                    // existing code...
                }
            }

            return ctx
        }
    }
})


```

frontend: App.vue

```html
<template>
  <router-view/>
  <MySnackbar ref="snackbarRef" baseIconError="fa fa-exclamation-triangle"></MySnackbar>
</template>

<script setup>

const snackbarRef = ref(null)

const executeSnackbar = (payload) => {
  const {message, type} = payload.detail
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

