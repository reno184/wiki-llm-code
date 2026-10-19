---
use-case: pdf-to-text-affichage-webp
version: 2
---

# Use case : affichage PDF → WebP (couche d'affichage du produit avocat)

Transformer les pages d'un PDF en images **WebP** pour un affichage rapide et maîtrisé (ex. PDF de 30 pages dans un carousel 3D Greensock) — au lieu d'un viewer PDF embarqué moche et lourd. Le PDF reste la **source de vérité** ; le WebP n'est qu'une couche de vue jetable et régénérable.

Brick du use case [pdf-to-text](README.md) : l'image affichée est aussi la **surface de sélection des zones** (ciblage), pas seulement une vignette.

## Pourquoi ça marche (validé, oct. 2026)

1. **Ça résout un vrai problème produit** : les viewers PDF embarqués (pdf.js, iframe, plugins) sont moches, lourds, lents à initialiser et cassent l'identité visuelle de l'app. Pour un avocat qui passe sa journée dans l'outil, afficher les pages comme des images natives dans une interface maîtrisée est un gain UX réel.
2. **Ça incarne la conviction ux-motion** ([front/ux-motion-adoption.md](../../front/ux-motion-adoption.md)) : un carousel 3D fluide sur les pages d'un contrat = le "wow" de démo qui vend l'adoption. La source de vérité reste le PDF, la vue est belle.
3. **La brique technique est validée** : **pdftocairo + sharp, côté Node** ([backend/pdftocairo-sharp.md](../../backend/pdftocairo-sharp.md)) rend les pages et encode le WebP (30-50% plus léger que PNG à qualité égale, parfait pour le web). Binaire `poppler-utils` dans l'image Docker de l'API, zéro pont inter-langue.
4. **Séparation vue/document propre** : le PDF reste la source de vérité (archivage, export, valeur probante), le WebP est régénérable à volonté — aucun risque de divergence, si le PDF change on régénère.

## Décision moteur (tranchée, oct. 2026)

- **Rendu côté Node** : pdftocairo (Poppler, remplace pdftoppm) + sharp, directement dans l'API Node.
- **Chemin Python écarté** (archivé pour ce besoin) : `pymupdf.get_pixmap` → WebP n'est plus le moteur de rendu d'affichage. PyMuPDF reste le moteur d'**extraction get_text** uniquement ([infra/pymupdf.md](../../infra/pymupdf.md)) ; `get_pixmap` garde son rôle de rendu de **zones vers LLM vision** (ocr-llm), pas d'affichage produit.
- Justification : pdf2image (Python) n'est qu'un wrapper du même Poppler → aucun gain à faire du Python pour du rendu, juste un micro-service en plus.

## Points à cadrer (en cours d'étude)

1. Zoom / lisibilité (pyramide de résolutions : blurhash base / dpi150 navigation / dpi300 ciblage — paramétrer le `-r` côté pdftocairo).
2. Cartographie des coordonnées Rect ↔ image (projection des zones ciblées sur l'affichage).
3. Perte recherche/sélection (assumée, à documenter noir sur blanc).
4. Génération : multi-résolutions, nommage ({docId}/p{NNN}_{level}.webp), lazy loading fenêtré, cache, invalidation par hash du PDF.

## État

**en construction** — moteur de rendu tranché et validé (pdftocairo + sharp côté Node) ; points restants : pyramide de résolutions, cartographie, nommage/cache.
