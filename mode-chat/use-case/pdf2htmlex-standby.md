# pdf2htmlEX — stand-by après essais non concluants (archivé)

Stack: pdf2htmlEX (conversion PDF → HTML fidèle), testé dans le labo Python (profil Python + Docker).
Usage: trace de décision — bibliothèque testée, non retenue pour l'instant, non exclue pour l'avenir.

## Décision

**Stand-by.** Ne pas investir davantage tant qu'aucune demande client réelle ne le justifie. Ne pas réutiliser par défaut dans les projets ; possibilité d'y revenir si un besoin concret apparaît (voir critères de réveil ci-dessous).

## Essais menés (2 tests non concluants)

### Test 1 — fiche de paie

- **Réussite** : export visuellement parfait d'une fiche de paie.
- **Échec** : les bordures des tableaux limitent la sélection — pas de sélection fluide des zones (ex. les congés payés). Le rendu est beau, mais l'exploitation est rigide.
- **Piste non explorée — mais limitée par la cause racine** : un retravailillage (post-processing du HTML généré) pourrait débloquer la sélection, MAIS le positionnement absolu est l'architecture même de l'outil : les chevauchements et la sélection rigide ne sont pas des bugs accidentels. Un retravailillage n'atteindra probablement jamais la fluidité recherchée.

### Test 2 — contrat PDF

- **Réussite** : reproduction HTML fidèle à ~95 %.
- **Échec** : des lignes se chevauchent, et aucun réglage ne corrige — plusieurs essais, changements de paramètres de la méthode, rien n'y fait : impossible d'atteindre un rendu 100 % parfait.
-Architecture obsolète et difficile : binaire C++ basé sur une ancienne version de Poppler, maintenance upstream quasi arrêtée. Dépendances obsolètes, déploiement douloureux sur Docker/macOS/Windows

## Conditions de réveil (quand réévaluer)

Une **demande client réelle** de conversion PDF → HTML sur des documents du même type (fiches de paie, contrats) — c'est le déclencheur principal.

## État

**archivé**

## tags

[rendu-realiste,absolute, convert-pdf, convert-html, python, pdf2htmlEX]