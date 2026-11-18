# vuejs-form — Formulaires Vue + DaisyUI

> Contrôles de formulaire daisyUI en Vue : snippets et pièges `v-model`.

Voir aussi [daisyui-components.md](daisyui-components.md) (composants daisyUI) et [../mode-agent/daisyui-component.md](../mode-agent/daisyui-component.md) (version agent).

## select avec required + option placeholder

```html
<select class="select" v-model="type" required>
  <option v-if="!type" :value="null" disabled selected>Model AI</option>
  <option value="qwen">Qwen AI</option>
  <option value="mistral">Mistral AI</option>
</select>
```

- `v-model="type"` (initialisé à `null` ou `""`) permet de gérer le `required` : tant que `type` est falsy, l'option placeholder reste sélectionnée et la validation HTML bloque la soumission.
- `v-if="!type"` sur l'option placeholder : dès que l'utilisateur choisit une valeur, le placeholder disparaît de la liste — il ne peut pas revenir sur le choix.
- `disabled` + `selected` : le placeholder s'affiche par défaut mais n'est pas sélectionnable une fois retiré.
- Adapter la valeur initiale au projet (`null` ou `""`, toujours falsy pour `v-if="!type"`).
