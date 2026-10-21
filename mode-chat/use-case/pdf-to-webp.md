Pas besoin de bench en live pour trancher — voici l'analyse pratique des deux stacks pour "PDF → images WebP" depuis une API.

## Comparatif

| Critère | Python `pymupdf` | Node + `pdftocairo` + sharp |
|---|---|---|
| Vitesse de rendu | Le plus rapide (moteur MuPDF, souvent 2–5× plus rapide que Poppler à DPI égal) | Correct, mais Poppler est plus lent par page |
| Sortie WebP | ❌ Pas natif : `pixmap.save()` ne sort que PNG/PNM/JPEG → il faut repasser par Pillow/cwebp (étape + I/O disque ou buffer) | ✅ Direct : sharp encode WebP nativement (libvips) |
| Overhead par appel | Lib chargée en mémoire, pas de fork | Spawn de process (`pdftocairo`) ~quelques ms + pipeline sharp (souvent 2 process si vous pipez via tmpfile/stdout) |
| Intégration API | Nécessite un microservice Python à côté de l'API Node | 100 % dans votre runtime Node ESM actuel |
| Parallélisme | Simple : pool de workers Python, pas de GIL pour le rendu C | Simple : `Promise.all` sur des pages, mais attention aux forks concurrents de `pdftocairo` |

**En synthèse** : PyMuPDF gagne sur le rendu brut, mais la perte WebP + l'ajout d'un service Python annulent souvent l'avantage. Pour une API Node existante, la stack pdftocairo + sharp est le choix pragmatique.

## Optimisations concrètes (stack Node)

1. **Une seule invocation par PDF** : `pdftocairo -png -r 150 prefix fichier.pdf` rend *toutes* les pages d'un coup — ne faites pas un spawn par page.
2. **Vérifiez votre version de Poppler** : les versions récentes supportent `pdftocairo -webp -q` directement → vous supprimez sharp et une étape complète. Testez `pdftocairo -h | grep webp`.
3. **Si sharp reste nécessaire** : pipez sans fichier temporaire. `pdftocairo -png -singlefile -f N -l N pdf.pdf -` sur stdout n'est pas supporté proprement pour `-png`, donc préférez `pdftocairo -png` vers un répertoire tmp (tmpfs de préférence), puis `sharp()` → `.webp({ quality: 80 })` → stream HTTP.
4. **DPI raisonnable** : `-r 150` suffit pour de la preview (le rendu domine le temps CPU ; passer de 300 à 150 DPI divise le temps par ~4).
5. **Cache** : clé `hash(pdf) + page + dpi + quality` sur disque/S3 — la vraie perf gagnante c'est de ne pas re-rendre.
6. **Limitez la concurrence** : un worker pool (p-limit à 2–4) sur le rendu ; forker 20 `pdftocairo` en parallèle sature le CPU et dégrade les latences P99.

## Quand basculer sur PyMuPDF

- Gros volumes batch (rendu dominant, WebP via Pillow `save(..., format="WebP")` en mémoire).
- PDF complexes/corrompus : MuPDF est nettement plus tolérant que Poppler.
- Dans ce cas, exposez-le en sidecar FastAPI appelé par Node, plutôt que de tout migrer.

Si vous voulez, je peux vous écrire le snippet Node ESM complet (spawn + sharp + gestion d'erreurs) — dites-le moi.
# pdftocairo + sharp — rendu PDF → WebP côté Node

Stack: Node.js (ESM) + binaire `pdftocairo` (Poppler, paquet `poppler-utils`) + sharp.
Usage: convertir les pages d'un PDF en images WebP directement depuis l'API Node.

## Décision (oct. 2026)

- Rendu d'affichage : **côté Node** (spawn pdftocairo + sharp), dans l'image Docker de l'API.
- **Poppler vit uniquement DANS le conteneur** : `apt-get install` s'exécute au **build de l'image Docker** (ligne RUN du Dockerfile), sur la base Debian de `node:22-slim`. Rien à installer sur le poste client (ni Linux, ni Mac, ni Windows) — le poste n'a besoin que de Docker. Le binaire `pdftocairo` n'existe pas à l'extérieur de Docker.

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
