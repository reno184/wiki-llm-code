# Due diligence M&A (code)

Sujet rare : audit technique d'un codebase/produit dans le cadre d'une acquisition (rachat de société, rachat d'actif digital). Objectif : estimer la dette technique, les risques et le coût de reprise AVANT de signer.

## Checklist « ne rien oublier »

| Zone | Points à vérifier | Signal rouge |
|---|---|---|
| Accès | Repo git complet, historique, CI/CD, hébergement, registre paquets, secrets manager | Refus d'accès repo = walk away |
| Secrets | Aucun secret en dur (clés API, BDD, SSH) dans le repo, même l'historique | Secret dans git history = rotation obligatoire post-deal |
| Dépendances | `npm audit` / `pip-audit`, paquets abandonnés, versions EOL | Failles critiques non patchées |
| Licences | Licences des dépendances (GPL dans un produit propriétaire = blocage) | Copyleft non détecté avant signature |
| Propriété | Contributeurs = employés/sous-traitants contractuels, pas de code copié d'Internet | Pas de CLA/contrat de propriété intellectuelle |
| BDD | Schéma, migrations, backups testés (restaurer un backup pour de vrai) | Pas de migration = schema figé dans le code |
| Environnements | prod ≠ dev, config par variables d'env (cf. [wiki-variable-env.md](wiki-variable-env.md)) | Config en dur, un seul environnement |
| Docs | README, setup local possible en <1h, runbooks | Impossible de lancer en local |
| Tests | Couverture réelle, tests qui passent, CI verte | 0 test, ou suite rouge ignorée |
| Dette | TODO/FIXME/HACK, dette mesurée, dead code | Ratio dette élevé = provision |
| Infra | IaC (Terraform/compose) vs clic-console | Infra invisible = recréer de zéro |
| Vendors | Comptes tiers (GCP, Stripe, domaines) transférables, pas personnels | Comptes perso du fondateur |

## Scripts d'analyse rapide

```bash
# 1. Secrets dans tout l'historique git
git log -p | grep -inE "(api[_-]?key|secret|password|token|BEGIN.*PRIVATE)" | head -50

# 2. Failles dépendances
npm audit --production 2>/dev/null; pip-audit 2>/dev/null

# 3. Dette technique
grep -rinE "TODO|FIXME|HACK|XXX" --include="*.{js,ts,py,vue}" . | wc -l

# 4. Licences copyleft
npx license-checker --summary  # ou pip install pip-licenses
```

```python
# Inventaire du repo : langages, taille, hotspots de complexité
import subprocess, collections, os
files = subprocess.run(["git","ls-files"], capture_output=True, text=True).stdout.split()
ext = collections.Counter(os.path.splitext(f)[1] for f in files)
big = sorted(((os.path.getsize(f), f) for f in files if os.path.exists(f)), reverse=True)[:10]
print(ext.most_common(10))
print(*big, sep="\n")  # les plus gros fichiers = hotspots à auditer en priorité
```

## Ordre de passage (1-2 jours)

1. Repo + setup local (si >1h de bloquer, noter).
2. Scripts ci-dessus (secrets, audit, dette, licences).
3. Lire les 10 plus gros fichiers + les 10 fichiers les plus modifiés (`git log --format= --name-only | sort | uniq -c | sort -rn | head`).
4. Interview des devs : qui sait quoi, qui peut partir.
5. Chiffrer : provision = jours de refactoring × TJM.

## Red flags contrats — la liste copier/coller

Liste brute pour relecture rapide de contrats (clients, fournisseurs, baux, prêts, licences). Destinée à servir de base de prompts/checklist pour une IA locale en due diligence M&A. Chaque item trouvé = creuser avant de signer, pas forcément walk away.

**Clauses déclenchées par la transaction**
- Change of control : prêt, bail ou licence qui devient exigible / résiliable au changement d'actionnariat → dette cachée à provisionner
- Clause d'assignment : contrat non cessible sans accord du cocontractant → les revenus peuvent ne pas suivre l'actif
- Droit de préemption / anti-dilution déclenché par l'entrée d'un investisseur

**Propriété intellectuelle**
- Devs freelance sans contrat de cession de droits (en France la cession n'est PAS automatique hors salariés) → tout ou partie du code n'appartient pas à la cible
- Contrat de développement communautaire/conjoint
- Marques/domaines déposés au nom du fondateur, pas de la société
- Dépendances copyleft non détectées (cf. checklist technique)

**Engagements financiers et passifs**
- Caution personnelle du fondateur solidaire des dettes
- Garantie de résultat / pénalités sans plafond
- Responsabilité illimitée / solidarité dans un contrat
- Dettes cachées : comptes courants d'associés, arrière de salaires/factures fournisseurs
- Abonnements auto-renouvelables longue durée avec préavis tardif (6-12 mois)

**Relation client / revenus**
- Client unique > 20-30% du CA → risque de perte à retournement
- Contrats verbaux ou side letters non comptabilisés
- Contrats à prix fixes avec périmètre mouvant (scope creep) sous-évalués
- Exclusivité accordée à un partenaire
- SLA impossibles à tenir (uptime 99,9% sans équipe on-call)

**Recherches textuelles (grep) que font les auditeurs dans les contrats**

```bash
# Français
grep -inE "changement de contrôle|cession|exclusiv|préavis|reconduction|tacite|pénal|garantie|solidar|illimité|caution|indemnis" *.pdf.txt

# Anglais
grep -inE "change of control|assignment|exclusiv|terminat|renewal|auto-renew|penalt|liability|indemnif|unlimited|personal guarantee|side letter" contracts/*.txt
```

- Rechercher aussi dans les emails/DocuSign « side letter », « excepté », « néanmoins » → aménagements hors contrat principal
- Croiser avec la checklist technique ci-dessus (secrets git, licences copyleft, comptes perso)

## Liens croisés

- [wiki-variable-env.md](wiki-variable-env.md) — config par variables d'env
- [wiki-git.md](wiki-git.md) — lecture d'historique git
- [wiki-secret-gcp.md](wiki-secret-gcp.md) — rotation des secrets post-deal
