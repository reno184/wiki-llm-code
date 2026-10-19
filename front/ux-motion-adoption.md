# UX-design & motion : l'esthétique et la fluidité comme levier stratégique d'adoption

Usage: conviction fondatrice, transversale à tous les produits (contrat, médecin, sites vitrines). À appliquer dès la conception de toute interface — pas une couche de finition ajoutée à la fin.

## La conviction

**Une application est utilisée si elle a un côté esthétique et du motion.** Une application belle, fluide, avec de belles transitions, est stratégique pour la vente, l'adoption et la rétention client — au-delà du service rendu.

Beaucoup de designers partagent ce constat : la qualité perçue, le plaisir d'usage et le motion design sont des **leviers commerciaux**, pas de la décoration. Le service rendu est la condition nécessaire ; l'expérience est ce qui fait choisir, revenir et recommander.

## Pourquoi ça marche (les mécanismes)

- **Qualité perçue = qualité du produit** : une interface soignée signale instantanément un produit sérieux et fiable — avant même la première fonction utilisée. Une interface moche fait douter du moteur derrière, même s'il est excellent.
- **Confiance** : pour des produits sensibles (contrats d'avocats, données de santé), la fluidité et le poli visuel rassurent sur le professionnalisme de l'outil — on confie rarement des documents confidentiels à une application qui paraît bricolée.
- **Motion = guidage** : les transitions (apparition/disparition, slide, feedback au clic) réduisent la charge cognitive : l'utilisateur comprend ce qui change, d'où vient le nouvel écran, ce qui a été validé. Le motion est de la pédagogie, pas de l'ornement.
- **Rétention** : le plaisir d'usage microscopique (le "feel") crée l'habitude. La rétention se joue sur le ressenti quotidien, pas sur la checklist de features.
- **Vente / démo** : dans une démo commerciale, l'impact des 30 premières secondes est visuel et émotionnel — une belle app se vend mieux parce qu'elle se montre mieux.

## Traduction concrète dans les produits

- **Ciblage Vue.js (pdf-to-text, avocats)** : l'ergonomie de sélection de zones est LE point d'usage — elle doit être fluide et satisfaisante (prévisualisation instantanée, feedback de sélection, transitions entre zoom/dézoom). Un outil de ciblage rigide tuera l'adoption même si l'extraction est parfaite.
- **Produit médecin** : l'interface de validation (human-in-the-loop) doit rendre la relecture agréable — c'est l'écran où l'utilisateur passe le plus de temps.
- **Sites vitrines** : View Transitions API + animations DaisyUI déjà dans la stack (voir front/vuejs-router-menu.md — transitions slide-fade entre routes).

## Règles pratiques

- Considérer le motion et l'esthétique **dès la conception** (wireframes), pas comme une passe finale.
- Transitions courtes et motivées (~150-300 ms) : guider, jamais retarder ni distraire.
- Cohérence : un système visuel unique (tokens, espacements, un vocabulaire d'animations), pas des effets au fil de l'eau.
- La performance fait partie de l'esthétique : une transition laggue détruit exactement ce qu'elle devait créer. (D'où l'attention aux poids d'images, lazy loading, etc.)

## Anti-patterns

- "On fera joli à la fin" : le refonte UX tardive coûte 10× plus cher que l'intégration dès le départ.
- Empiler les features en négligeant le ressenti : la checklist ne retient personne, l'expérience oui.
- Motion gratuit : chaque animation doit avoir une raison (guidage, feedback, hiérarchie).

## État

**vivant** — conviction transversale, appliquée au ciblage Vue.js du use case pdf-to-text et aux produits médecin/avocat.
