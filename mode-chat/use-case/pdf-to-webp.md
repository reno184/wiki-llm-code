# webp
 
## Résumé du pipeline

Ces trois éléments forment une chaîne de traitement **PDF → image → WebP** dans un conteneur Node.js :

**1. Dockerfile — installation des dépendances système**
```dockerfile
RUN apk add --no-cache poppler-utils
```
- Installe **Poppler** (via le gestionnaire de paquets d'Alpine Linux) sans garder le cache des paquets (image Docker plus légère).
- Fournit notamment l'outil en ligne de commande **`pdftocairo`**, qui convertit les pages PDF en images (PNG/JPEG).

**2. Node.js — appel du processus de conversion**
```js
import { spawn } from 'node:child_process'
const proc = spawn('pdftocairo', args)
```
- Exécute `pdftocairo` depuis Node.js comme un processus externe (non bloquant, flux gérés).
- Avec les bons `args` (ex : `-png`, `-r 150`, page, préfixe de fichier), il transforme une page du PDF en image PNG dans `tmpDir`.

**3. Node.js — compression en WebP**
```js
import sharp from 'sharp'
const buffer = await sharp(path.join(tmpDir, filename)).webp({ quality }).toBuffer()
```
- **Sharp** lit l'image temporaire générée, la convertit au format **WebP** avec un niveau de compression défini par `quality`, et retourne le résultat en `Buffer` (prêt à être renvoyé via HTTP, stocké en base, etc.).

### En une phrase

**`pdftocairo` convertit une page du PDF en image PNG, puis sharp compresse cette image en WebP pour obtenir un fichier léger et optimisé pour le web.**


## État

**vivant** 

## tags 

[nodejs, convert-image, convert-pdf]
