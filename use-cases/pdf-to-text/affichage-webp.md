---
use-case: pdf-to-text-affichage-webp
version: 1
---

# Use case : affichage PDF → WebP (couche d'affichage du produit avocat)

Transformer les pages d'un PDF en images **WebP** pour un affichage rapide et maîtrisé (ex. PDF de 30 pages dans un carousel 3D Greensock) — au lieu d'un viewer PDF embarqué moche et lourd. Le PDF reste la **source de vérité** ; le WebP n'est qu'une couche de vue jetable et régénérable.

Brick du use case [pdf-to-text](README.md) : l'image affichée est aussi la **surface de sélection des zones** (ciblage), pas seulement une vignette.

## Pourquoi ça marche (validé, oct. 2026)

1. **Ça résout un vrai problème produit** : les viewers PDF embarqués (pdf.js, iframe, plugins) sont moches, lourds, lents à initialiser et cassent l'identité visuelle de l'app. Pour un avocat qui passe sa journée dans l'outil, afficher les pages comme des images natives dans une interface maîtrisée est un gain UX réel.
2. **Ça incarne la conviction ux-motion** ([front/ux-motion-adoption.md](../../front/ux-motion-adoption.md)) : un carousel 3D fluide sur les pages d'un contrat = le "wow" de démo qui vend l'adoption. La source de vérité reste le PDF, la vue est belle.
3. **La brique technique est déjà validée** : `page.get_pixmap(dpi=…)` ([infra/pymupdf.md](../../infra/pymupdf.md)) rend les pages ; WebP n'est que l'encodage de sortie (30-50% plus léger que PNG à qualité égale, parfait pour le web).
4. **Séparation vue/document propre** : le PDF reste la source de vérité (archivage, export, valeur probante), le WebP est régénérable à volonté — aucun risque de divergence, si le PDF change on régénère.

## Points à cadrer (en cours d'étude)

1. Zoom / lisibilité (pyramide de résolutions).
2. Cartographie des coordonnées Rect ↔ image (projection des zones ciblées sur l'affichage).
3. Perte recherche/sélection (assumée, à documenter noir sur blanc).
4. Rendu côté Node ou Python (micro-service, moteur).
5. Génération : multi-résolutions, nommage, lazy loading, cache.

## État

**en construction** — les 4 points "pourquoi ça marche" sont actés ; les 5 points à cadrer sont en cours d'étude approfondie.
