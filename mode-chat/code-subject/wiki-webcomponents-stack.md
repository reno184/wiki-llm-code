# wiki-webcomponents-stack — stack micro-framework : Lit, signals, TanStack Router

Stack cible pour sites à forte manipulation DOM et calculs de position, alternative à Vue quand la réactivité d'un framework complet devient illisible à grande échelle. Zéro snippet ici : décisions, pièges, règles de bascule.

## Philosophie

Coquille applicative minimaliste + briques visuelles intensives en vanilla JS adaptées de CodePen via LLM, documentées JSDoc. Lit (~5 Ko) comme « le framework le moins framework » : réactivité **par composant** uniquement (champ `@property` → re-render du composant seul), jamais de graphe réactif global — c'est ce qui répond au problème « je ne comprends plus rien quand l'app grossit ».

## Lit

- Toujours créer les web components AVEC Lit, jamais `customElements.define` brut (3× plus de code, pièges de constructeur) sauf exigence client.
- Portabilité totale : le web component est utilisable partout, y compris dans un futur projet Vue/React d'un client. Lock-in quasi nul (standards web).
- Shadow DOM = encapsulation CSS naturelle… mais frontière assumée avec le monde extérieur (voir pièges slots et modules vanilla).

## Signals

- Un signal = une valeur réactive ; c'est la couche de notification d'état de TOUTE la stack (composants Lit, contexts, routeur).
- **Factory functions + closures pour les contexts** (`createAuth()`, `createCart()`) : pattern module éprouvé, mais une closure n'est PAS réactive — chaque factory doit aussi exposer un signal ou un `subscribe()`, sinon on réinvente la réactivité de Vue en moins bien.
- Règle d'hygiène : état vraiment privé en closure, état partagé exposé dans un signal/signal lisible (une closure pure est invisible en debug, pas de DevTools).
- Dès que 2-3 contexts ont des dépendances croisées : adopter UNE lib de signals unique pour toute l'app (ex. signal de Preact ~1 Ko, compatible Lit et TanStack) au lieu de DIY.
- WeakMap obligatoire dans tout mécanisme de subscription maison, pour éviter les fuites/listeners fantômes.

## TanStack Router — oui ou non ?

Problème de TanStack : il exige un système réactif — c'est le seul point où la stack impose une contrainte (à prévoir dès le départ, pas en cours de route).

**Signals NE remplace PAS un routeur** (routing = synchronisation URL ↔ état : history.pushState, parsing params, guards, preload), MAIS un mini-routeur à base de signals suffit pour un site simple, et signe l'indépendance vis-à-vis de la lib.

**Règle de bascule — mini-routeur signals DIY tant que :**
- peu de pages, routes plates ou faiblement imbriquées ;
- pas de search params complexes à typer/partager entre routes ;
- pas de guards d'auth par branche ni de preload.

**Basculer vers TanStack Router dès que :**
- search params structurés = source de vérité typée sérialisée (filtres, pagination, tri — le killer feature de TanStack) ;
- routes imbriquées + guards + layouts composés s'accumulent ;
- le coût mental du DIY dépasse ~14 Ko de lib.

**Layouts imbriqués en DIY** : chaque segment de route = un composant Lit ; le layout reste MONTÉ pendant que son enfant change (c'est ça, le vrai routing imbriqué : préserver l'état du layout). Prévoir une interface unique (`navigate`, `route`, `segments`) derrière laquelle on peut brancher TanStack plus tard sans toucher aux composants.

## Les pièges des slots (le point de friction n°1 de Lit)

1. **On ne peut PAS remplir un slot nommé depuis un template Lit parent** : tout contenu passé via le template Lit va dans le slot par défaut (attribut `slot` non supporté sur les fragments). Parade : contenu par props (render prop, ex. `.child=${() => html…}`) ou consommateur en light DOM statique.
2. **Styling du contenu slotté : `::slotted` obligatoire** — le Shadow DOM ne style pas le light DOM. Et seul le premier niveau est ciblable (`::slotted(nav a)` ne marche pas) ; le style interne du contenu reste à son composant d'origine.
3. **Contenu slotté ≠ props réactives** : réagir au changement via `@slotchange` + `assignedElements({ slot: … })`/`assignedNodes()`.
4. **Fallback du slot par défaut** : le contenu dans `<slot>` ne s'affiche QUE si rien n'est slotté (ce n'est pas un préfixe) ; fallback partiel à gérer en JS.
5. **Les slots ne transfèrent pas la réactivité** : un composant Lit slotté se re-render seul (ok), mais du HTML pur mis à jour à l'extérieur est invisible pour le layout sans `slotchange`.

**Règle de décision** : slots natifs pour l'API publique du composant (contrat standard web component, consommé par du HTML externe/autre framework) ; render props pour la composition interne app Lit → Lit (plus flexible, sans le piège n°1).

## Modules visuels intensifs (canvas, drag, calculs de position)

- Classes vanilla JS autonomes montées dans un composant Lit : reçoivent un élément + options, exposent `destroy()`, ne savent rien du reste de l'app.
- Snippets CodePen adaptés par LLM + JSDoc complet.
- `destroy()` dans `onBeforeUnmount`/disconnectedCallback : retrait des listeners (addEventListener manuel = fuite mémoire sinon).
- Attention Shadow DOM : les snippets qui font `querySelector` global ne trouvent plus rien → utiliser `renderRoot`/`this.shadowRoot`, ou composer en light DOM.

## Liens croisés

- [wiki-purejs.md](wiki-purejs.md) — Proxy object, réactivité from scratch (le moteur commun)
- [vue-dom-manipulation.md](../use-case/vue-dom-manipulation.md) — le monde réactif vu de Vue (nextTick, click.stop, pièges v-if)
- [wiki-vue-props-reactivity.md](wiki-vue-props-reactivity.md) — comparaison avec la réactivité Vue props
