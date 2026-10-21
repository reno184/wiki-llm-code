# wiki-webstorm-http — client HTTP WebStorm : réutiliser la réponse d'une requête dans la suivante

Usage: fichier `.http` dans WebStorm — enchaîner des requêtes sans copier/coller à la main la valeur retournée par un upload (ex. `group` renvoyé par le step1, réutilisé dans le step2). Jamais chargé par un agent.

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

## Pièges à ne pas oublier

- **Ordre d'exécution** : exécuter les requêtes dans l'ordre (icône play à gauche de chaque `###`). `{{group}}` est vide tant que le step1 n'a pas été joué → URL `.../step1/` → 404/erreur.
- **Portée des variables** : `client.global.set` = persistant pour toute la session du fichier. `client.set` = variable locale (même valeur par défaut dans tous les environnements mais pas enregistrée globalement). Pour un id chaîné d'une requête à l'autre, `client.global` est le bon choix.
- **Vérifier la valeur stockée** : le panneau Variables (icône en haut à droite du fichier `http`) liste toutes les variables globales et leur valeur courante — dépannage immédiat quand `{{group}}` semble vide.
- **Réinitialiser** : `client.global.clearAll()` dans un handler, ou bouton reset des variables du panneau, si une valeur obsolète persiste entre deux sessions.
- **Environnements** : pour host/port, préférer un env `dev` (`{{host}}` + fichier `http-client.env.json`) plutôt que des variables globales — `client.global` est réservé aux valeurs **dynamiques** issues des réponses.
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

**vivant** — pattern utilisé sur le projet anonymisation (chaînage step1 upload PDF → step1 GET pages par `group`, oct. 2026).
