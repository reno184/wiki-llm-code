# wiki-python — Déboguer du Python (IDE, points d'arrêt, debug requêtes Flask)

Fiche `mode-chat/code-subject` — usage ~1×/mois, copié/collé manuel.

## Points d'arrêt Python : dans quel IDE ?

- **WebStorm ne supporte PAS le débogage Python** : le plugin Python est réservé à IntelliJ IDEA Ultimate et PyCharm. Pas de run/debug configuration Python dans WebStorm.
- Alternatives validées :
  - **PyCharm Community (gratuit)** : point d'arrêt en cliquant dans la gouttière, `Run → Debug`, variables + call stack + console interactive.
  - **VS Code** : extension **Python** (Microsoft), point d'arrêt dans la gouttière, `F5`, `launch.json` si args.
  - **IntelliJ IDEA Ultimate** (si licence) : ajouter le plugin Python, IDE familial JetBrains avec débogueur complet.
- Sans IDE : `breakpoint()` (Python 3.7+) ou `import pdb; pdb.set_trace()`. Commandes : `n` next, `s` step into, `c` continue, `p var` print, `q` quit.

## Section débogage — inspecter une requête Flask entrante (form + fichiers)

Snippet à coller en début de handler pour voir ce que le serveur reçoit réellement :

```python
print("=== REQUEST FORM ===", flush=True)
print(request.form.to_dict(), flush=True)

# Afficher les fichiers reçus (ex: file)
print("=== REQUEST FILES ===", flush=True)
print(request.files.to_dict(), flush=True)
```

### Pourquoi ce snippet
- `request.form.to_dict()` → tous les champs texte du multipart (y compris les blocs JSON envoyés comme champs).
- `request.files.to_dict()` → les fichiers reçus avec leur nom de champ (`file`, `file[]`…).
- Piège classique : champ attendu dans `form` mais envoyé dans `files` (ou inverse) → snippets vides côté handler. Ces deux prints tranchent immédiatement.

### Piège `flush=True` (à ne pas retirer)
- Sans `flush=True`, stdout est **bufférisé** : dans un conteneur Docker ou derrière gunicorn/uvicorn, les prints n'apparaissent qu'à la fin du process (ou jamais dans `docker logs`). Toujours garder `flush=True` pour du debug temps réel.

### Checklist
- [ ] Snippet collé en tout début de handler, AVANT tout parsing
- [ ] `flush=True` sur chaque print
- [ ] Retirer les prints avant commit (debug uniquement)
- [ ] Côté client : vérifier le `Content-Type` du bloc (un champ `Content-Type: application/json` arrive dans `form` en texte brut, pas parsé)

## Liens croisés
- [webstorm/](webstorm/) — fiches WebStorm (config, HTTP client, agents) : WebStorm pour JS/TS, jamais pour déboguer Python.
- [wiki-webstorm-http.md](wiki-webstorm-http.md) — tester ces mêmes requêtes depuis WebStorm (client HTTP `.http`, multipart, champs JSON).
- [upload-file-busboy.md](upload-file-busboy.md) — équivalent parsing multipart côté Express/busboy.
