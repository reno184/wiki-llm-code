# pdf2htmlEX — stand-by après essais non concluants (archivé)

Stack: pdf2htmlEX (conversion PDF → HTML fidèle), testé dans le labo Python (profil Python + Docker).
Usage: trace de décision — bibliothèque testée, non retenue pour l'instant, non exclue pour l'avenir.

## Décision

**Stand-by.** Ne pas investir davantage tant qu'aucune demande client réelle ne le justifie. Ne pas réutiliser par défaut dans les projets ; possibilité d'y revenir si un besoin concret apparaît (voir critères de réveil ci-dessous).

## Essais menés (2 tests non concluants)

### Test 1 — fiche de paie

- **Réussite** : export visuellement parfait d'une fiche de paie.
- **Échec** : les bordures des tableaux limitent la sélection — pas de sélection fluide des zones (ex. les congés payés). Le rendu est beau, mais l'exploitation est rigide.
- **Piste non explorée** : un retravailillage (post-processing du HTML généré, ajustement des bordures/tableaux) pourrait débloquer la sélection — à retester si besoin réel.

### Test 2 — contrat PDF

- **Réussite** : reproduction HTML fidèle à ~95 %.
- **Échec** : des lignes se chevauchent, et aucun réglage ne corrige — plusieurs essais, changements de paramètres de la méthode, rien n'y fait : impossible d'atteindre un rendu 100 % parfait.

## Conditions de réveil (quand réévaluer)

- Une **demande client réelle** de conversion PDF → HTML sur des documents du même type (fiches de paie, contrats) — c'est le déclencheur principal.
- Ou l'apparition d'une alternative qui résout le problème de sélection dans les tableaux / le chevauchement de lignes.

Si réveil : repartir des deux essais ci-dessus comme baseline, cibler d'abord le chevauchement de lignes (le plus bloquant : aucun paramètre ne l'a corrigé).

## Anti-patterns

- Réutiliser pdf2htmlEX par défaut sur un nouveau projet : rendu ~95 % = non conforme pour un livrable client.
- Repousser l'essai "pour voir si ça marche mieux" sans demande derrière : du temps de labo sans décision au bout.

## État

**archivé (stand-by)** — décision 2026-10 : deux essais non concluants, attente d'une demande client réelle. Hors INDEX (politique _archive/), cette fiche sert uniquement de trace.
