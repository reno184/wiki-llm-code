# Upload de fichier — front (input type file + FormData + fetch)

Stack: JS natif (ou Vue 3), fetch, FormData.
Usage: sélectionner un fichier côté navigateur, le valider, l'envoyer au backend via multipart/form-data. Fiche front du trio upload fullstack (voir [upload-file-busboy.md](upload-file-busboy.md) et [upload-strategy.md](upload-strategy.md)).

## Formulaire — HTML

```html
<form id="uploadForm">
  <input type="file" id="fileInput" accept="application/pdf,image/*" />
  <button type="submit" id="uploadBtn">Envoyer</button>
  <p id="uploadError" class="hidden text-error"></p>
  <p id="uploadProgress"></p>
</form>
```

- `accept` : filtre la boîte de dialogue côté OS — **confort UX uniquement**, ne remplace jamais la validation serveur.

## Envoi — JS

```js
const MAX_SIZE = 20 * 1024 * 1024 // 20 Mo — même limite que le back
const ALLOWED_TYPES = ['application/pdf', 'image/jpeg', 'image/png', 'image/webp']

const form = document.getElementById('uploadForm')
const input = document.getElementById('fileInput')
const btn = document.getElementById('uploadBtn')
const errEl = document.getElementById('uploadError')

form.addEventListener('submit', async (e) => {
  e.preventDefault()
  errEl.classList.add('hidden')

  const file = input.files[0]
  if (!file) return showError('Aucun fichier sélectionné.')
  if (!ALLOWED_TYPES.includes(file.type)) return showError('Type de fichier non autorisé.')
  if (file.size > MAX_SIZE) return showError('Fichier trop volumineux (max 20 Mo).')

  const formData = new FormData()
  formData.append('file', file) // ⚠️ toujours un Blob/File, jamais un string
  formData.append('category', 'material') // champs annexes = côté server, lus par busboy (fields)

  btn.disabled = true
  try {
    const rep = await fetch('/api/upload', {
      method: 'POST',
      body: formData, // pas de Content-Type manuel : le navigateur pose le boundary multipart
    })
    if (!rep.ok) {
      const { message } = await rep.json().catch(() => ({ message: 'Erreur inconnue' }))
      return showError(message || `Erreur ${rep.status}`)
    }
    const { url, key } = await rep.json()
    console.log('Uploadé :', url, 'clé :', key)
  } catch (err) {
    showError('Erreur réseau pendant l\'upload.')
  } finally {
    btn.disabled = false
  }
})

function showError(msg) {
  errEl.textContent = msg
  errEl.classList.remove('hidden')
}
```

## Vue 3 + DaisyUI — pattern input file

```html
<div class="form-control">
    <input type="file" class="file-input file-input-primary w-full" accept=".pdf,application/pdf" @change="handleChange">
</div>
```

```js
const handleChange = (event) => {
    const files = event.target.files
    if (!files || files.length === 0) return

    const formData = new FormData()
    formData.append('file', files[0])
    bodyData.value = formData
}
```

- `event.target.files` = `FileList`, même sans `multiple` — **le comptage n'est jamais garanti côté serveur**, d'où la validation back.
- `accept=".pdf,application/pdf"` : double notation (extension + MIME) pour couvrir tous les OS/navigateurs.

### Contrat single/multiple — l'attribut `multiple` décide

**Sans `multiple` sur l'input, c'est un upload single file — côté front ET côté back.**

- **Sans `multiple`** : le sélecteur OS n'autorise qu'un fichier, `files` contient au plus 1 élément → on `append` `files[0]` seul.
- **Avec `multiple`** : boucler sur la `FileList` :

```js
for (const file of files) {
    formData.append('file', file)
}
```

Ce choix est **structurant pour toute la chaîne** (politique 400/500, voir la fiche busboy) :

| Input front | Back attendu | Erreur "fichier multiple sur route single" |
|---|---|---|
| sans `multiple` | un seul fichier | improbable → **500** |
| avec `multiple` | N fichiers (`Promise.all`) | — |

Le back contrôle `uploadedFile !== null` → 500 (fiche busboy) **parce que** le front fixe explicitement single ou multiple. Modifier l'attribut d'un côté sans l'autre = incohérence de contrat.

## Notes

- **Ne jamais set `Content-Type` à la main** avec FormData : c'est le navigateur qui génère le header multipart avec son `boundary`. Un header manuel (même `multipart/form-data`) casse le parsing busboy côté back.
- Valider type + taille **côté client ET côté serveur** : le client évite un aller-retour inutile, le serveur est la seule vraie garantie (le front est contournable).
- `input.files[0]` = `File` (hérite de Blob) : s'appaire directement avec `formData.append('file', file)`.
- Multi-fichiers : `<input type="file" multiple>` puis `for (const f of input.files) formData.append('files', f)` — busboy émettra un event `file` par fichier.
- Si reCAPTCHA v3 protège la route, le token va dans le header `g-recaptcha-response` (voir [recaptcha-invisible-v3.md](recaptcha-invisible-v3.md)).
- En Vue 3, même logique : `<input type="file" @change="onFileChange">`, garder le `File` dans un `ref`, pas de v-model natif sur un input file.

## État

**vivant** — pattern d'entrée d'un upload fullstack, aligné avec la fiche busboy (mêmes limites MAX_SIZE / ALLOWED_TYPES).
