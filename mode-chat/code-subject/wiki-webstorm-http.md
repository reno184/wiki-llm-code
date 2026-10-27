# wiki-webstorm-http — client HTTP WebStorm : réutiliser la réponse d'une requête dans la suivante

Usage: fichier `.http` dans WebStorm — enchaîner des requêtes sans copier/coller à la main la valeur retournée par un upload (ex. `group` renvoyé par le step1, réutilisé dans le step2), déclarer des variables in-place (`@titleBulk`) et uploader plusieurs fichiers en une requête. Jamais chargé par un agent.

## Le principe

Le client HTTP de WebStorm (fichiers `.http`) permet d'exécuter du JS **après** chaque requête via un handler de réponse. On y stocke une valeur de la réponse dans une **variable globale**, puis on l'injecte dans la requête suivante avec `{{variable}}`. Zéro copier/coller, zéro erreur de frappe sur un id généré dynamiquement.

## Le pattern complet

```http
### step1 convert pdf to webp
POST http://localhost:3000/anonymisation/step1
Content-Type: multipart/form-data; boundary=WebStormBoundary

--WebStormBoundary
Content-Disposition: form-data; name="file"; filename="racine_5_suite_1.pdf"
Content-Type: application/pdf

< ./mock/racine_5_suite_1.pdf

--WebStormBoundary--

> {%
    client.global.set("group", response.body.group);
%}

### step1 pages d'un groupe (ordre carousel)
GET http://localhost:3000/anonymisation/step1/{{group}}
```

Les 3 briques :

1. **Handler de réponse** : le bloc `> {% ... %}` juste après la requête. `response.body` = le JSON parsé de la réponse. On y fait `client.global.set("group", response.body.group)`.
2. **Référence dans la requête suivante** : `{{group}}` est remplacé par la valeur stockée au moment de l'exécution.
3. **Séparateur** `###` : chaque requête du fichier est délimitée par un titre `###`, toutes vivent dans le même fichier `requests.http` (ou autre nom `.http`).

## Variables in-place : déclarer une valeur réutilisable dans le fichier

Syntaxe `@nom = valeur` **au-dessus de la requête** (ligne seule, avant le `###` ou le verbe HTTP). Portée = le fichier `.http` courant ; la variable est immuable (définie une seule fois, pas écrasable). On l'utilise ensuite avec `{{nom}}`.

```http
@titleBulk = Bulletins 2022
```

Différences avec `client.global.set` :

| | In-place `@nom =` | `client.global.set` |
|---|---|---|
| Définie | en dur dans le fichier | à l'exécution (handler JS) |
| Portée | ce fichier `.http` | session, tous fichiers `.http` |
| Écrasable | non (immuable) | oui (`client.global.set` / `clearAll`) |
| Usage | config statique (titre bulk, host) | valeur dynamique d'une réponse (id, token) |

## Upload multi-fichiers + variable in-place

Plusieurs blocs `file` avec le **même** `name="file"` dans le même multipart → le back reçoit un tableau de fichiers. On combine avec la variable in-place pour le champ texte.

```http
@titleBulk = Bulletins 2022

### step1 convert pdf to webp
POST http://localhost:3000/api/import/bulk
Content-Type: multipart/form-data; boundary=WebStormBoundary

--WebStormBoundary
Content-Disposition: form-data; name="titleBulk"

{{titleBulk}}

--WebStormBoundary
Content-Disposition: form-data; name="file"; filename="22_01_BULLETIN.pdf"
Content-Type: application/pdf

< ./mock/22_01_BULLETIN.pdf
--WebStormBoundary
Content-Disposition: form-data; name="file"; filename="22_02_BULLETIN.pdf"
Content-Type: application/pdf

< ./mock/22_02_BULLETIN.pdf
--WebStormBoundary
Content-Disposition: form-data; name="file"; filename="22_03_BULLETIN.pdf"
Content-Type: application/pdf

< ./mock/22_03_BULLETIN.pdf
--WebStormBoundary--

> {%
    client.global.set("group", response.body.group);
%}

### step1 pages d'un groupe (ordre carousel)
GET http://localhost:3000/anonymisation/step1/{{group}}
```

À noter sur le multi-fichiers :

- **Même `name`** pour chaque bloc fichier (`name="file"`) — sinon le back ne les voit pas comme un tableau.
- **Ordre des blocs = ordre d'upload** : garder l'ordre naturel des fichiers (01, 02, 03…) pour un traitement séquentiel côté back.
- Un champ texte (`titleBulk`) s'intercale entre les blocs fichiers sans casser le multipart : la valeur est la ligne après `Content-Disposition` (ici `{{titleBulk}}` résolu en `Bulletins 2022`).
- **Blank line obligatoire** entre le `Content-Disposition` d'un fichier et son contenu `< ./chemin` — sans elle, le fichier part en pièce jointe vide ou corrompue.
- Le `--WebStormBoundary--` final ferme le multipart, un seul pour toute la requête.

## Pièges à ne pas oublier

- **Portée des variables** : `client.global.set` = persistant pour toute la session du fichier. `client.set` = variable locale (même valeur par défaut dans tous les environnements mais pas enregistrée globalement). Pour un id chaîné d'une requête à l'autre, `client.global` est le bon choix.
- **In-place variable immuable** : `@titleBulk` ne peut être redéfinie dans le même fichier ; pour une valeur par campagne qui change souvent, la modifier à la ligne `@` directement.
- **Vérifier la valeur stockée** : le panneau Variables (icône en haut à droite du fichier `http`) liste toutes les variables globales et leur valeur courante — dépannage immédiat quand `{{group}}` semble vide.
- **Réinitialiser** : `client.global.clearAll()` dans un handler, ou bouton reset des variables du panneau, si une valeur obsolète persiste entre deux sessions.
- **Fichier binaire** : la syntaxe `< ./chemin/relatif.pdf` envoie le fichier en binaire ; le chemin est relatif au fichier `.http`.
- **Réponse non-JSON** : `response.body` est le JSON parsé ; si le back renvoie du texte brut, utiliser `response.body` comme string telle quelle.

## Variantes utiles

```http
### stocker un header de la réponse (ex. token)
> {%
    client.global.set("token", response.headers.valueOf("Authorization"));
%}

### l'utiliser ensuite
GET http://localhost:3000/api/me
Authorization: Bearer {{token}}
```

- Assertion rapide dans le même handler : `client.test("201", () => response.status === 201)` — visible dans l'onglet Tests.

## État

**vivant** — pattern utilisé sur le projet anonymisation (chaînage step1 upload PDF → step1 GET pages par `group`, oct. 2026) et sur le projet import bulk (multi-fichiers `Bulletins 2022` + variable in-place `@titleBulk`, oct. 2026).
