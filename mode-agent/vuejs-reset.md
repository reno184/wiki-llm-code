# vuejs-reset — page d'erreur 500 d'un projet Vue

## Règle 1

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
