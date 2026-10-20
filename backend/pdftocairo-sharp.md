---
name: pdftocairo-sharp
layer: backend
version: 1
---

# pdftocairo + sharp — rendu PDF → WebP côté Node

Stack: Node.js (ESM) + binaire `pdftocairo` (Poppler, paquet `poppler-utils`) + sharp.
Usage: convertir les pages d'un PDF en images WebP directement depuis l'API Node — couche d'affichage du use case [pdf-to-text/affichage-webp](../use-cases/pdf-to-text/affichage-webp.md). **Remplace pdftoppm** (pdftocairo = rendu Cairo, antialiasing et fidélité meilleurs) et **écarte la lib Python** (pdf2image n'est qu'un wrapper de Poppler ; PyMuPDF get_pixmap n'est plus chargé du rendu — PyMuPDF reste cantonné à l'extraction get_text, voir [libraries.md](../libraries.md)).

## Décision (oct. 2026)

- Rendu d'affichage : **côté Node** (spawn pdftocairo + sharp), dans l'image Docker de l'API.
- **Poppler vit uniquement DANS le conteneur** : `apt-get install` s'exécute au **build de l'image Docker** (ligne RUN du Dockerfile), sur la base Debian de `node:22-slim`. Rien à installer sur le poste client (ni Linux, ni Mac, ni Windows) — le poste n'a besoin que de Docker. Le binaire `pdftocairo` n'existe pas à l'extérieur de Docker.
- Extraction : toujours le micro-service Python PyMuPDF get_text — l'unicité "un seul moteur rendu+extraction" est abandonnée, assumée : rendu Node, extraction Python.
- Une lib Python n'apporte rien ici : le moteur de rendu serait le même binaire Poppler, avec un pont inter-langue en plus.

## Dockerfile (backend)

```dockerfile
FROM node:22-slim

RUN apt-get update \
    && apt-get install -y --no-install-recommends poppler-utils poppler-data \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
USER node
CMD ["node", "src/server.js"]
```

- `poppler-utils` fournit pdftocairo ; `poppler-data` = polices/encodages CJK (garder si PDF clients internationaux, sinon optionnel).
- `apt-get` tourne dans le conteneur au build, pas sur le poste hôte ; `rm -rf /var/lib/apt/lists/*` évite de garder le cache apt dans l'image.
- Vérifier une fois déployé : `docker exec <conteneur> pdftocairo -v` (doit afficher la version).
- sharp embarque ses binaires libvips prébuilds — rien à installer pour lui.

## Code validé (ESM, Règle 0 : zéro commentaire)

```js
import {spawn} from 'node:child_process'
import fs from 'node:fs/promises'
import os from 'node:os'
import path from 'node:path'
import sharp from 'sharp'

export async function convertPdfToWebp(pdfBuffer, {firstPageOnly = false, quality = 80} = {}) {
    const tmpDir = await fs.mkdtemp(path.join(os.tmpdir(), 'pdf-convert-'))
    const tempPdfPath = path.join(tmpDir, 'input.pdf')
    const outputPrefix = path.join(tmpDir, 'page')

    try {
        await fs.writeFile(tempPdfPath, pdfBuffer)

        const args = ['-png', '-r', '150']

        if (firstPageOnly) {
            args.push('-f', '1', '-l', '1')
        }

        args.push(tempPdfPath, outputPrefix)

        await new Promise((resolve, reject) => {
            const proc = spawn('pdftocairo', args)
            const errorChunks = []

            proc.stderr.on('data', (chunk) => errorChunks.push(chunk))
            proc.on('error', reject)
            proc.on('close', (code) => {
                if (code !== 0) {
                    return reject(new Error(`pdftocairo a échoué (code ${code}): ${Buffer.concat(errorChunks).toString()}`))
                }

                resolve()
            })
        })

        const files = await fs.readdir(tmpDir)
        const pageNumberOf = (file) => parseInt(file.match(/\d+/)?.[0] ?? '0', 10)
        const pngFiles = files
            .filter((file) => file.startsWith('page-') && file.endsWith('.png'))
            .sort((a, b) => pageNumberOf(a) - pageNumberOf(b))

        if (pngFiles.length === 0) {
            throw new Error(`pdftocairo n'a généré aucune page`)
        }

        const webpBuffers = await Promise.all(
            pngFiles.map(async (filename) => {
                const pngBuffer = await fs.readFile(path.join(tmpDir, filename))
                return sharp(pngBuffer).webp({quality}).toBuffer()
            })
        )

        return firstPageOnly ? webpBuffers[0] : webpBuffers
    } finally {
        await fs.rm(tmpDir, {recursive: true, force: true}).catch((err) => {
            console.error('Nettoyage dossier temporaire échoué:', err)
        })
    }
}
```

## Notes / pièges

- `-r 150` = dpi de rendu ; pour la pyramide dpi150 (navigation) / dpi300 (ciblage), paramétrer le dpi.
- Tri naturel obligatoire (`page-10` avant `page-2` en lexicographique) — regex sécurisée `(match(/\d+/)?.[0] ?? '0')`.
- `firstPageOnly` → `-f 1 -l 1` : une seule page rendue, retour Buffer seul au lieu de Buffer[].
- Garde `pngFiles.length === 0` : un PDF corrompu peut sortir code 0 sans page générée.
- `spawn` sans shell : les arguments ne passent jamais par un shell, pas d'injection sur les paths/args.
- `finally` + `fs.rm(force: true)` : le tmpdir est toujours nettoyé, même en échec.
- À durcir si input utilisateur direct : timeout sur le process (spawn + kill) et limite de pages (`-l` max).
- `Promise.all` sur toutes les pages d'un gros PDF = RAM ; batcher ou séquentialiser au-delà de ~30 pages.

## État

**vivant** — validé oct. 2026 (proto pdf-to-text) ; code durci vs première version (tri safe, garde pages vides, handler `error` du spawn).
