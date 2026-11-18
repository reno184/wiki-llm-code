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


### Point de vigilence VPS dimention du CPU

**Mon avis sur le 1 vCPU / 2 Go / 20 Go pour ton use case :**

✅ **Ça passe, avec des réserves à connaître :**
- **pdftocairo** (poppler) est un binaire externe : chaque conversion est un processus séparé, léger en RAM (~50-150 Mo par PDF standard). Très raisonnable.
- **sharp** est le point d'attention : il fait ses décodages/encodages en RAM via libvips. Sur des images A4 à 150-300 DPI, compte quelques dizaines de Mo par image en traitement — ça tient sur 2 Go si tu ne traites pas en parallèle.

⚠️ **Les vraies limites de cette offre :**
1. **Le CPU est le goulot, pas la RAM.** Un vCPU = les conversions s'enchaînent : compte quelques secondes par page PDF. Si tu lances 3 conversions simultanées, tout ralentit.
2. **20 Go de disque** : vérifie la taille de tes PDF et des WebP générés, + le système + Docker (~2-4 Go). Si tu stockes les fichiers sur le serveur, ça peut vite saturer — prévois un nettoyage ou un stockage objet (Public Storage Infomaniak) si le volume grandit.
3. **Pas de swap garanti** : ajoute un fichier swap (1-2 Go) dès l'installation pour éviter un OOM kill si sharp dépasse ponctuellement.

**Recommandations pratiques :**
- Limite la concurrence à 1-2 conversions simultanées dans ton backend (file d'attente simple) et mets un timeout sur le `spawn`.
- Ajoute `--no-sandbox`-free... non, pour pdftocairo rien de spécial, mais mets `ulimit -v` ou une limite mémoire Docker (`mem_limit` dans compose) pour qu'un PDF corrompu ne fasse pas tomber le serveur.
- Teste ton pipeline complet en local avec `docker stats` pour valider la RAM réelle avant de choisir définitivement.

Bref : bon choix pour démarrer, et si les conversions deviennent fréquentes, tu passes à l'échelon au-dessus en quelques minutes.

Feu vert quand tu veux pour la création du VPS.

## État

**vivant** 

## tags 

[nodejs, convert-image, convert-pdf]
