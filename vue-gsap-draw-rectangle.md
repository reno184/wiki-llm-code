---
title: GSAP (GreenSock) dans Vue - overlay rectangle de sélection animé
tags: [frontend, howto]
fiches-liees: [vue-dom-manipulation]
---
# GSAP + Vue : overlay rectangle de sélection

## Cas d'usage

Plusieurs blocs sélectionnables sur un PDF affiché dans un conteneur. Un overlay div suit la bounding box englobante des blocs sélectionnés, animé avec GSAP pour un effet de dessin fluide (draw motion).

## Le computed : bounding box englobante

```js
const boundingBox = computed(() => {
    if (selectedBlocksMap.value.size === 0 || !pdfContainer.value) return null

    const containerRect = pdfContainer.value.getBoundingClientRect()

    const rectsList = Array.from(selectedBlocksMap.value.values())
        .map(block => block.rect)

    const lefts = rectsList.map(r => r.left)
    const rights = rectsList.map(r => r.right)
    const tops = rectsList.map(r => r.top)
    const bottoms = rectsList.map(r => r.bottom)

    const minX = Math.min(...lefts)
    const maxX = Math.max(...rights)
    const minY = Math.min(...tops)
    const maxY = Math.max(...bottoms)

    return {
        left: minX - containerRect.left,
        top: minY - containerRect.top,
        width: maxX - minX,
        height: maxY - minY
    }
})
```

## Le watch : animation GSAP de l'overlay

```js
watch(boundingBox, (newBox) => {
    if (newBox && selectedIds.value.length > 0) {
        gsap.to(selectionOverlay.value, {
            duration: 0.12,
            display: 'block',
            left: newBox.left,
            top: newBox.top,
            width: newBox.width,
            height: newBox.height,
            ease: 'power2.out'
        })
    } else {
        gsap.to(selectionOverlay.value, {
            duration: 0.1,
            display: 'none',
            width: 0,
            height: 0
        })
    }
})
```

Durations courtes (0.1 / 0.12 s) : l'overlay suit la sélection sans traîner. `power2.out` pour un départ vif qui freine.

## Points d'attention

### Pas de `{deep: true}` sur ce watch

Le computed `boundingBox` recrée un nouvel objet à chaque recalcul de ses dépendances réactives (`selectedBlocksMap`, `pdfContainer`). Le watcher détecte déjà le changement de référence : `deep` ajoute un parcours récursif inutile à chaque tick.

### `getBoundingClientRect()` dans un computed n'est PAS réactif

L'appel DOM ne dépend d'aucune donnée réactive : scroll, resize ou zoom du conteneur ne re-déclenchent PAS le recalcul du computed. La bounding box devient fausse après un scroll tant qu'une autre dépendance réactive ne change pas. Si l'overlay doit suivre le scroll/resize, forcer explicitement :

```js
// écouter scroll/resize et forcer la mise à jour
useEventListener(pdfContainer, 'scroll', () => triggerRef(selectedBlocksMap))
useEventListener(window, 'resize', () => triggerRef(selectedBlocksMap))
```

`triggerRef()` (Vue 3) force la re-évaluation sans muter les données.

### Animations concurrentes

`gsap.to` répété sur le même élément peut empiler des tweens si la sélection change très vite. En pratique les durations de 0.1 s rendent le chevauchement invisible ; sinon ajouter `overwrite: 'auto'` dans l'objet de config.
