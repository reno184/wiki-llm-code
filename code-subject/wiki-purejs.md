# wiki-purejs — JS natif sans framework

Connaissances et patterns JavaScript pur, sans framework. Pas de dépendance.

## Proxy object — réactivité from scratch

### Principe

Un `Proxy` enveloppe un objet et intercepte ses opérations via des *traps* (`get`, `set`, `deleteProperty`, `has`…). En appelant une fonction de mise à jour du DOM dans le trap `set`, toute affectation rafraîchit le DOM automatiquement. **C'est exactement le moteur de la réactivité de Vue 3.**

### Snippet minimal

```js
function reactif(etat, majDom) {
  return new Proxy(etat, {
    set(cible, prop, valeur) {
      cible[prop] = valeur
      majDom()
      return true
    },
  })
}

const etat = reactif({ compteur: 0 }, () => {
  document.querySelector('#compteur').textContent = etat.compteur
})

etat.compteur = 5   // DOM mis à jour tout seul
```

### Réactivité imbriquée (proxy dans le trap get)

Un Proxy n'est PAS profond par défaut : les objets enfants doivent être re-wrappés à la lecture.

```js
const proxifies = new WeakMap()   // anti re-proxy infini

function reactifProfond(etat, majDom) {
  if (proxifies.has(etat)) return proxifies.get(etat)
  const p = new Proxy(etat, {
    get(cible, prop) {
      const v = cible[prop]
      return (v && typeof v === 'object') ? reactifProfond(v, majDom) : v
    },
    set(cible, prop, valeur) {
      cible[prop] = valeur
      majDom()
      return true
    },
  })
  proxifies.set(etat, p)
  return p
}
```

### Ce que Vue ajoute par-dessus (les limites du pur Proxy)

- **Tracking fin** : Vue sait *quelle* propriété est lue par *quel* effet (dependency tracking dans `get`) et ne re-render que le nécessaire — le snippet maison recharge tout.
- **Batching** : plusieurs mutations = un seul patch DOM, puis `nextTick` (cf. [vue-dom-manipulation.md](../use-case/vue-dom-manipulation.md)).
- **Clés à gérer à la main** : tableaux (index), `deleteProperty`, `has`.
- **Nouveaux objets attribués** (`etat.user = {...}`) : à wrapper soi-même (le trap `set` ci-dessus ne le fait pas).
- **WeakMap obligatoire** pour ne pas re-proxyer un objet déjà wrappé (boucle infinie sinon).

### Liens croisés

- [vue-dom-manipulation.md](../use-case/vue-dom-manipulation.md) — nextTick, click.stop, pièges v-if + querySelector (le monde réactif vu de Vue)
- [wiki-vue-props-reactivity.md](wiki-vue-props-reactivity.md) — la réactivité de Vue appliquée aux props
