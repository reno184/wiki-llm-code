# WebStorm HTTP Client — tester un upload multipart (.http)

Stack: WebStorm HTTP Client natif (fichiers `.http`/`.rest`), API Express + busboy.
Usage: tester une route d'upload multipart/form-data sans Postman ni front — directement dans l'IDE, avec fichiers réels. Fiche IDE du trio upload (voir [../backend/upload-file-busboy.md](../backend/upload-file-busboy.md) et [../front/upload-file.md](../front/upload-file.md)).

## Principe

Le HTTP client de WebStorm (intégré, gratuit) lit des fichiers `.http` à la racine du projet (ex. `requests.http`). Chaque bloc séparé par `###` = une requête exécutable via le gutter vert. Variables `{{host}}` définies dans `http-client.env.json` (non commité s'il contient des tokens).

Un upload multipart s'écrit **à la main** : header `Content-Type: multipart/form-data; boundary=<Boundary>` puis le body avec les sections délimitées par `--<Boundary>`.

## Anatomy d'un upload dans .http

```
POST {{host}}/api/studyMaterial/newNovel
Authorization: Bearer {{token_iam_gcloud}}
Content-Type: multipart/form-data; boundary=WebStormBoundary

--WebStormBoundary
Content-Disposition: form-data; name="title"

test

--WebStormBoundary
Content-Disposition: form-data; name="file"; filename="test_page_1.png"
Content-Type: image/png

< ../test_files/screenshot_page_1.png

--WebStormBoundary--
```

Points critiques :

- **`boundary=WebStormBoundary`** : déclaré dans le header, répété dans le body (`--WebStormBoundary` avant chaque partie, `--WebStormBoundary--` en toute fin). Les deux doivent correspondre exactement, sinon busboy ne parse rien (cf. [uploadValidator](../backend/upload-file-busboy.md)).
- **Champs texte** : une section par champ — `name="title"` puis la valeur sur la ligne suivante. C'est ce que busboy émet dans `on('field')`.
- **Fichier** : `name="file"` = le nom du champ attendu par la route, `filename="test_page_1.png"` = le nom **vu par le serveur** — utile pour tester les regex de nommage (ex. `_page_1` pour PAGE_NUMBER, voir fiche busboy).
- **`< ../test_files/screenshot_page_1.png`** : syntaxe du HTTP client pour **inclure un fichier binaire** — chemin relatif au fichier `.http`. C'est le corps réel envoyé au serveur.
- **Ligne vide obligatoire** entre les headers de section et le contenu.

## Exemple 1 — POST avec champs + fichier (newNovel)

```
### -------------------------------------------------------------------------
### ÉCRAN : Page study material add new novel
### COMPOSANT VUE : `src/views/StudyMaterialPage.vue`
### DESCRIPTION : Permet d'ajouter novel (stratégies différentes pour short stories, webarticle)
### STRATEGY :
###   Novel strategy -> fields title, author -> plusieurs images regex _page_1
###   Short stories strategy -> fields title, author, titleShortStory -> plusieurs images regex _page_1
###   Web article strategy -> fields title, articleUrl -> une seule image pas de regex
###
### Attention il faut neutraliser les IA si test car payment (extract OCR, resumeNovel, etc...)
### ocrProcessor, resumeEnglishNovelPrompt, resumeShortStoriesPrompt
### -------------------------------------------------------------------------

POST {{host}}/api/studyMaterial/newNovel
Authorization: Bearer {{token_iam_gcloud}}
x-test-uid: {{test-uid}}
Content-Type: multipart/form-data; boundary=WebStormBoundary

--WebStormBoundary
Content-Disposition: form-data; name="title"

test

--WebStormBoundary
Content-Disposition: form-data; name="author"

testdsfs ddd

--WebStormBoundary
Content-Disposition: form-data; name="file"; filename="test_page_1.png"
Content-Type: image/png

< ../test_files/screenshot_page_1.png

--WebStormBoundary--
```

## Exemple 2 — PUT upload sur une route à params (updateNovel)

Même syntaxe pour PUT — l'ID dans l'URL, un seul fichier, pas de champs :

```
### -------------------------------------------------------------------------
### ÉCRAN : Page study material (novel) update tile
### COMPOSANT VUE : `src/views/StudyMaterialPage.vue`
### DESCRIPTION : Cette route aurait pu se situer dans studyMaterial (control regex _page_1)
### short stories ne peut pas ajouter d'images après coup car le résumé de l'oeuvre
### se fait sur le texte OCR uniquement à la création
### params : studyMaterialId
### Attention test désactiver IA ocrProcessor
### -------------------------------------------------------------------------

PUT {{host}}/api/sentenceContext/updateNovelWithNewImages/RzfS9g4hwFTJcnhE1id7
Authorization: Bearer {{token_iam_gcloud}}
x-test-uid: {{test-uid}}
Content-Type: multipart/form-data; boundary=WebStormBoundary

--WebStormBoundary
Content-Disposition: form-data; name="file"; filename="test_page_1.png"
Content-Type: image/png

< ../test_files/screenshot_page_1.png

--WebStormBoundary--
```

## Environnement — `http-client.env.json`

```
{
  "dev": {
    "host": "http://localhost:3000",
    "token_iam_gcloud": "<token>",
    "test-uid": "<uid>"
  }
}
```

- `{{host}}`, `{{token_iam_gcloud}}`, `{{test-uid}}` : résolus à l'exécution, sélecteur d'env en haut du fichier `.http`.
- **Tokens non commités** : le vrai `http-client.env.json` va dans `.gitignore`, committer un `http-client.env.json.example` sans valeurs (même logique que `.env` — voir [nodejs-express-env.md](../backend/nodejs-express-env.md)).

## Notes

- Le **gros avantage** du `.http` sur Postman : les requêtes sont **versionnées avec le projet** (repo git) et relisibles par l'agent/Junie — il sait comment tester chaque route sans deviner le format.
- Les blocs de commentaires `### ...` au-dessus d'une requête documentent l'écran Vue, la stratégie, les IA à désactiver — c'est de la doc de test vivante.
- Header custom (`x-test-uid`) : fonctionne comme n'importe quel header, utile pour bypasser l'auth en environnement de test.
- Pour tester le **contrôle de poids** (400, fiche busboy) : envoyer un fichier > `limitMo` et vérifier la popover/réponse 400.
- Pour tester les **regex de nommage** (fiche busboy, FILENAME_PATTERNS) : jouer sur `filename="..."` — `test_page_42.png`, `2026-06-12_vocabulary.csv`, `vocabulary_Sheet1.xlsx` — sans créer de nouveaux fichiers.
- Multi-fichiers : répéter la section `name="file"` avec des `filename` différents.

## État

**vivant** — pattern de test des routes d'upload dans WebStorm, aligné avec les fiches busboy (validation) et upload front.
