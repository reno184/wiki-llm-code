# Labo Python Docker — environnement d'entraînement isolé

Stack: Python 3.x dans Docker (macOS), manipulation images PNG / PDF / OCR / conversion Markdown / création de PDF.
Usage: environnement de labo pour tester des bibliothèques Python ; Docker privilégié plutôt qu'un environnement virtuel pour isoler Python sur le Mac.

## Pourquoi Docker plutôt qu'un venv sur macOS

- **Isolation système complète** : les bibliothèques de manipulation (OCR, PDF, images) ont souvent des dépendances binaires natives (tesseract, libpng, poppler, imagemagick, cairo). Dans un venv, ces dépendances se retrouvent installées globalement sur le Mac (Homebrew), mélangées aux mises à jour système, et cassent silencieusement à chaque upgrade macOS. Docker embarque ces binaires dans l'image : le Mac reste vierge.
- **Reproductibilité** : un `docker compose up` sur une machine neuve reconstitue le labo exactement (versions Python, binaires, libs) — un venv ne reproduit que les packages pip, pas les dépendances système.
- **Tests à grande échelle sans conséquence** : c'est un labo — on installe, casse, désinstalle, change de version d'une lib pour comparer. Chaque expérience ratée se jette avec le conteneur ; rien ne suinte dans les autres projets.
- **Snapshots** : on peut figer une image (`docker commit`) avec un état précis de libs pour y revenir, ou avoir plusieurs images parallèles (une par écosystème de libs testées) — impossible avec un seul venv.

## Usage type

- Suivre la fiche `infra/dockerfile-python.md` pour l'image (base slim, requirements.txt, USER non-root).
- `docker compose run --rm pipeline` pour lancer un script isolé jetable (voir compose type de la fiche).
- Une branche/l'installation de lib expérimentale se fait dans l'image, validée ou rejetée, puis requirements.txt mis à jour — jamais directement sur le Mac.

## Anti-patterns

- Installer tesseract/poppler/etc. via Homebrew "juste pour tester" sur le Mac : pollution système, dérive entre machines, c'est ce que Docker évite.
- Monter le venv du Mac dans le conteneur (incompatibilité de chemins, plateformes différentes).
- Une seule image fourre-tout qui mélange toutes les libs testées : préférer une image par domaine (OCR, PDF, images) quand les dépendances entrent en conflit.


## todo

- Après proto, faire le grand ménage des libs par exmple fitz remplacer par pymupdf, ne pas mettre :latest car problème de compatiblité, quelle lib en javascript, les lib python, le requirements le apt get, le catch, les performances. bref il faut que dans un an je puisse comprendre tout des libs, que j'ai utilisé pour convertir pdf (anonymisation), extraction, OCR, convertion image, slection text d'un pdf scanné, langchain llm local...

## État

**vivant** — décision validée 2026-10 : Docker systématique, venv réservé aux rares cas où une lib native est impossible à conteneuriser.
