---
title: WebStorm - gitignore .idea en conservant les runConfigurations
tags: [outils, decision]
fiches-liees: []
---

# WebStorm : exclure .idea sauf les run configurations

## Décision

Par défaut, tout le dossier `.idea/` (méta-données WebStorm) est exclu du dépôt. Exception : `.idea/runConfigurations/`. Il contient les configurations de debug Front-end et Back-end, et il est hyper pratique de les versionner pour ne pas les recréer à chaque checkout.

## Le piège : `.idea/` casse la réinclusion

La syntaxe évidente **ne fonctionne pas** :

```gitignore
.idea/
!.idea/runConfigurations/
```

Git ne peut pas réinclure un fichier si un de ses répertoires parents est exclu avec un slash final. `runConfigurations` est dans `.idea/` qui est exclu, donc la négation est ignorée silencieusement.

## La bonne config

Exclure le *contenu* de `.idea` avec `.idea/*` (pas le dossier lui-même), puis réinclure le sous-dossier :

```gitignore
.idea/*
!.idea/runConfigurations/
```

Ainsi `workspace.xml`, `usage.statistics.xml` et le reste restent ignorés, mais les fichiers `*.run.xml` des configurations de debug Front-end / Back-end sont commités.

## Vérification

```bash
git check-ignore -v .idea/runConfigurations/Node_Debug.run.xml   # ne doit rien afficher
git check-ignore -v .idea/workspace.xml                          # doit afficher la règle .idea/*
```
