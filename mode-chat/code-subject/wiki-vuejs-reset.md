# wiki-vuejs-reset — Reset/init projet Vue : shell HTML min-h-screen + page Erreur 500

## Règle 1 — Shell index.html : `min-h-screen` sur `body` ET `#app` (pattern à répéter à chaque nouveau projet Vue)

Dans `index.html`, poser systématiquement :

```html
<html lang="en" data-theme="light">
<body class="min-h-screen bg-linear-to-b from-gray-100 to-gray-300">
    <div id="app" class="min-h-screen"></div>
```

**Pourquoi `min-h-screen` sur `body` ET `#app`** : garantit que le background (dégradé `bg-linear-to-b` ici) s'étend sur toute la hauteur même quand on scrolle. Sans ça, le background s'arrête à la hauteur du viewport initial et le bas de page devient blanc au scroll. À répéter sur tout nouveau projet Vue.

**Piège** : mettre le dégradé seulement sur `#app` ou seulement sur `body` ne suffit pas selon le layout des vues — le pattern validé = les deux.

**Pourquoi `data-theme="light"` sur `<html>`** : bloque l'application en thème clair, indépendamment de la configuration macOS (mode sombre du système). Sans cet attribut, les composants DaisyUI suivent `prefers-color-scheme` et l'app passe en thème sombre sur un mac en dark mode. `lang="en"` (ou `fr`) posé au passage sur la même balise.

## Règle 2 — Page d'erreur 500

Chercher dans le dossier `/views` le composant `Error500Page.vue`. S'il n'existe pas, le créer avec le code suivant :

```vue
<template>
  <div class="min-h-screen flex items-center justify-center p-4">
    <div class="w-full max-w-2xl">
      <!-- Alert Component -->
      <div class="alert alert-error items-start shadow-lg p-6 ">
        <div class="w-full">
          <template v-if="isDev">
            <h3 class="font-bold text-lg mb-2">
              Erreur Serveur (Vue Développeur)
            </h3>
            <pre class="bg-error-content bg-opacity-10 p-4 rounded text-sm overflow-x-auto whitespace-pre-wrap text-white">{{ errorStack }}</pre>
          </template>
          <template v-else>
            <h3 class="font-bold text-lg mb-1">
              Erreur 500
            </h3>
            <p class="text-sm opacity-90">
              Oups, une erreur s'est produite... Notre équipe a été notifiée.
            </p>
          </template>
        </div>
      </div>
      <div class="flex justify-center mt-6">
        <a href="/" class="btn btn-primary">
          Retourner à l'accueil
        </a>
      </div>
    </div>
  </div>
</template>

<script setup>
import {computed, ref} from 'vue'

defineOptions({name: 'Error500Page'})

const isDev = computed(() => window.location.hostname === 'localhost')

const errorStack = ref(history.state?.stack || 'Aucune stack trace disponible.')
</script>
```
