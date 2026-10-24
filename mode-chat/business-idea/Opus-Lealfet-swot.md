Rôle : tu es un consultant en stratégie produit et go-to-market spécialisé dans les
logiciels de sécurité civile et de gestion de crise en France (SDIS, communes,
préfectures, gendarmerie). Tu connais le fonctionnement des achats publics locaux,
la culture des sapeurs-pompiers volontaires, les contraintes CNIL/RGPD et la
réalité du terrain (zone rurale sans 4G, téléphone bas de gamme).

Contexte produit — lis attentivement, tout l'analyse doit s'appuyer dessus :
Je suis développeur fullstack freelance (TJM 470 € HT, 15 ans d'expérience).
Je prépare un projet OPEN SOURCE : une PWA (Progressive Web App) de cartographie
tactique tempérée d'alertes géolocalisées, sans backend obligatoire.

Stack et capacités techniques déjà maîtrisées :
- Leaflet.js, Turf.js (détection temps réel d'un point entrant dans une zone :
  booleanPointInPolygon, pointToLineDistance), Node.js, WebSockets, PWA installable.
- Fonctionne hors-ligne (tuiles OSM pré-téléchargées), sans store d'application,
  sans installation — crucial pour les téléphones bas de gamme des volontaires.
- Innovation clé : une zone dessinée sur la carte est encodée en ~35 caractères
  (polyline encodée type Google) et transmissible par simple SMS copier-coller
  (ex: "Z1:kaggGoyw_@{c@o}@nKw|AvcAf^kk@~{B"). Le destinataire colle le SMS dans
  l'app, la zone s'affiche. Cela fonctionne sans data, sans serveur, sans abonnement.
- API d'alerte : WhatsApp Business Cloud, email, push PWA.
- Serveur MCP prévu : zones et alertes exposées comme outils pour agents IA.
- Sources de données publiques : NASA FIRMS (points de chaleur satellites),
  vigilances Météo-France, cadastre/IGN.
- Traitement temps réel de plusieurs milliers de points à 10 Hz côté serveur
  (matching Turf contre les zones, pré-filtre bbox, calcul en worker Node,
  envoi au client limité au viewport + alertes).
- Conformité : pas de tracking d'individus ni de reconnaissance faciale ;
  la géolocalisation navigateur ne fonctionne qu'app au premier plan ;
  vigilance RGPD/CNIL sur toute donnée personnelle.

Offres de service cibles (proposition de valeur terrain) :
1. Périmètre de feu / zonage d'évacuation (CODIS -> chefs de groupe via SMS).
2. Recherche de personne disparue (RPS) : découpage de zone en secteurs assignés.
3. Zones interdites / points d'eau / DZ hélico partagés en une trame SMS.
4. Zonage temporaire militaire (tir, manœuvre) et briefing reconnaissance.
5. Alertes automatiques : point de chaleur FIRMS entrant dans un polygone ->
   notification WhatsApp/email/push.

Modèle économique envisagé : open source + formation à l'intégration (2 j à
~940 € HT) + abonnement hébergé (50–150 €/mois) + missions d'intégration MCP/SI
au TJM. Marché d'entrée : sapeurs-pompiers (SDIS, communes), puis gendarmerie/militaire.

Ta mission — produis les livrables suivants, chacun clairement structuré,
chiffré quand c'est possible, et ancré dans la réalité française (achats publics,
culture pompiers volontaires, contraintes CNIL) :

1. SWOT complet du projet (strengths, weaknesses incl. l'absence de business
   récurrent au départ, opportunities, threats incl. les acteurs en place),
   sous forme de tableau + 5 actions prioritaires qui en découlent.

2. BUSINESS MODEL : dresse le Business Model Canvas (9 blocs). Définis les 3-4
   segments de clientèle prioritaires avec leur canal d'achat réel (marché public,
   convention, budget propre association). Propose une grille tarifaire
   formation / abonnement / intégration, et le chemin de conversion entre les
   niveaux. Chiffre le potentiel : nombre de SDIS/communes adressables en France.

3. WEBINAR / VISIBILITÉ : un plan concret pour me faire connaître en 90 jours :
   webinaire (titre, accroche, plan des 45 min, plateforme, audience cible),
   5 canaux complémentaires (LinkedIn, presse spécialisée type Professions
   Secours, fédérations FNSPF, événements type Salon Préventica...), et un
   calendrier semaine par semaine.

4. ASSURANCES / MUTUELLES / ÉLUS : comment impliquer les élus locaux et les
   collectivités (financeurs naturels), et identifier les financeurs annexes
   (mutuelles des pompiers, assureurs locaux type Maif/Macif/Maif qui
   financent la prévention des risques). Donne l'angle d'approche et l'argumentaire
   en 3 points pour chacun.

5. CONCURRENCE : liste et analyse des solutions concurrentes ou adjacentes en
   France et en Europe (outils SIG de SDIS, applications de géolocalisation
   d'alertes, plateformes de gestion de crise type Astroguard/GEOSIS/Crisis
   Room...) : forces, faiblesses, positionnement prix, et mon avantage
   différenciant face à chacune. Signale les risques d'un acteur institutionnel
   (intégration par l'Etat ou un éditeur public).

6. ADOPTION PWA : analyse les freins et leviers d'adoption d'une PWA dans les
   milieux pompiers/gendarmerie/militaire (confiance dans une app sans store,
   politique de sécurité des téléphones professionnels, iOS vs Android),
   et propose des palliatifs (mode démo, export PDF, version native en veille).

7. PERFORMANCE : recommande une architecture temps réel robuste pour plusieurs
   milliers de points à 10 Hz (ingestion, worker Turf, pré-filtre bbox, WebSocket
   vs SSE, backpressure), avec les seuils où changer de technologie (ex: Redis,
   spatial index type R-tree) et les métriques à monitorer.

8. CNIL / RGPD : checklist de conformité précise pour ce projet (base légale des
   traitements, durée de conservation des traces GPS, données des volontaires,
   registre des traitements, impact du mode hors-ligne sur la localisation),
   et les 3 interdits absolus à documenter dans le README pour se protéger.

Contraintes de forme : français, ton direct et opérationnel, pas de langue de
bois, chiffré quand possible, tableaux en Markdown, et termine par un plan
d'action "30 premiers jours" ordonné par impact/effort.


# Réponse Opus - PWA de cartographie tactique — Analyse stratégique & Go-to-Market
## Consultant produit / achats publics sécurité civile — France

> **Avertissement de méthode** : les chiffres marqués *(est.)* sont des estimations de marché à valider. Les chiffres institutionnels (nombre de SDIS, communes, effectifs) sont des ordres de grandeur publics 2024-2025. Trois affirmations de votre brief sont factuellement fragiles et je les attaque dans le texte : **NASA FIRMS comme source d'alerte opérationnelle**, **le 10 Hz / milliers de points**, et **l'abonnement à 50-150 €/mois**.

---

# 1. SWOT

## 1.1 Tableau

| | **POSITIF** | **NÉGATIF** |
|---|---|---|
| **INTERNE** | **STRENGTHS**<br><br>**S1 — Le SMS-zone (~35 car.) est une vraie innovation défendable.** C'est le seul élément du projet qu'aucun concurrent n'a. Il répond à une douleur réelle et documentée : le chef de groupe en zone blanche, le SPV avec un Android à 90 €, la DZ hélico à transmettre en 10 s. Tout le reste de la stack est commoditisé.<br><br>**S2 — Zéro installation, zéro store, zéro compte.** Contourne le MDM, le DSI, le store Apple/Google, le RGPD "création de compte". Le time-to-value est < 60 s.<br><br>**S3 — Open source = contournement de l'objection n°1 du secteur public** : la réversibilité et la pérennité d'un éditeur unipersonnel. L'AGPL + dépôt du code chez le client neutralise "et si vous disparaissez ?".<br><br>**S4 — Hors-ligne réel (tuiles pré-téléchargées).** 100 % des concurrents SaaS tombent en zone blanche. Vous êtes le seul debout là où le besoin est maximal.<br><br>**S5 — Coût de structure quasi nul.** Pas de levée, pas de salarié, TJM de repli à 470 €. Vous pouvez tenir 24 mois d'itération là où une startup financée doit scaler en 18.<br><br>**S6 — Créneau politique ouvert** : post-incendies Gironde 2022, pacte capacitaire, Beauvau de la sécurité civile, pression "souveraineté + open source" (circulaire État plateforme, DINUM). L'open source français est un argument qui ouvre des portes en 2026. | **WEAKNESSES**<br><br>**W1 — Zéro revenu récurrent au démarrage, et le modèle n'en crée pas mécaniquement.** Open source + formation one-shot + intégration au TJM = **vous êtes un freelance avec un portfolio**, pas un éditeur. La valeur de sortie est nulle. *Donc : il faut un objet récurrent contractualisé dès le client n°1 (voir §2).*<br><br>**W2 — L'abonnement à 50-150 €/mois est une erreur de catégorie.** Une collectivité ne sait pas acheter à 600 €/an : le coût administratif d'un bon de commande + service fait + mandatement est de 150 à 400 € en charge interne. Vous êtes sous le seuil de visibilité budgétaire. *Donc : passer à des paliers annuels 490 € / 1 200 € / 3 500 € / 9 000 € / 25 000 €.*<br><br>**W3 — Formation 2 j à 940 € HT = vous vous payez 470 €/jour pour animer un groupe.** C'est votre TJM de prestation individuelle appliqué à une activité à forte valeur collective. Le marché CNFPT/ENSOSP/privé est à 1 100-1 900 € HT **par stagiaire** sur 2 j, ou 4 500-7 000 € HT en intra pour 8-12 personnes. *Donc : ×5 sur le prix formation.*<br><br>**W4 — Pas de Qualiopi = pas de financement OPCO/CNFPT/FPT.** Sans certification, une collectivité ne peut pas imputer votre formation sur son plan de formation avec les mêmes facilités. *Donc : soit Qualiopi (4-8 k€, 3-6 mois), soit vous renommez "formation" en "prestation d'accompagnement au déploiement" et vous la vendez en marché de services.*<br><br>**W5 — Risque "énième couche cartographique".** Un chef de groupe a déjà : le SGO sur tablette, l'ATLAS/SIG départemental, Géoportail, parfois Esri Field Maps. Votre app est le 4ᵉ écran. En intervention, le 4ᵉ écran ne s'ouvre pas. *Donc : positionnez-vous explicitement comme "le plan B quand les trois autres ne marchent pas", pas comme un outil de commandement.*<br><br>**W6 — NASA FIRMS est inutilisable en détection opérationnelle en France.** VIIRS/MODIS : résolution 375 m, 2-4 passages/jour, latence NRT ~ 3 h (l'URT à 60 s est réservée USA/Canada). À H+3 un feu de forêt méditerranéen fait déjà 20-80 ha et le CODIS a été alerté par le 18 depuis 2 h 50. *Donc : ne JAMAIS vendre FIRMS comme détection. Repositionner en veille grande échelle / suivi de front / usage ONF-DFCI-exploitants forestiers-assureurs, ou usage outre-mer et international.*<br><br>**W7 — Le 10 Hz / plusieurs milliers de points n'a pas de cas d'usage.** 200 agents en opération × 1 position / 10 s = **20 événements/s**. FIRMS France = quelques centaines de points / jour. Vous avez dimensionné pour 1 000× le besoin. *Donc : supprimer cette exigence du cahier des charges, elle coûte des semaines et ne se vend pas (voir §7).*<br><br>**W8 — Homme seul = disqualifiant sur marché formalisé.** Beaucoup de SDIS exigent des références, une RC Pro, un PAS/PSSI, parfois une homologation. *Donc : RC Pro éditeur + cyber (1 500-3 500 €/an) et un groupement avec un intégrateur régional dès le mois 3.*<br><br>**W9 — Aucune référence, aucun retour d'expérience, aucun uniforme.** Le milieu SP achète à des gens qui ont porté la tenue. *Donc : vous DEVEZ recruter un co-porteur officier SP (actif ou retraité) comme caution. Non négociable.*<br><br>**W10 — Responsabilité en cas d'erreur de zone.** Une zone mal décodée = un binôme envoyé du mauvais côté d'un front de feu. *Donc : CGU + bandeau produit "outil d'aide, non substituable à l'ordre verbal du COS", et checksum sur la trame SMS.* |
| **EXTERNE** | **OPPORTUNITIES**<br><br>**O1 — Loi Matras (25/11/2021) + décret 2022-907 : explosion de l'obligation PCS/PICS.** Le nombre de communes soumises au Plan Communal de Sauvegarde est passé d'environ 11 000 à **~16 000-20 000** *(est.)*, et les EPCI concernés doivent élaborer un PICS. Toutes ont besoin de **cartographier des zones** (zones inondables, points de rassemblement, secteurs de référents, itinéraires d'évacuation) et de les **diffuser à des bénévoles sans budget IT**. C'est votre marché le plus liquide.<br><br>**O2 — Les AASC et les CCFF : le vrai point d'entrée oublié.** Associations agréées de sécurité civile (Protection Civile, Croix-Rouge, FFSS, Croix Blanche, SNSM, équipes cynotechniques) = **plusieurs milliers d'unités locales** avec budget propre, achat libre sans marché public, besoin quotidien de zonage (DPS, battues, recherche de personne). Les Comités Communaux Feux de Forêts (PACA/Corse/Occitanie, **plusieurs centaines**) patrouillent en zone blanche avec leurs téléphones perso. C'est un marché à décision en 2 semaines et à cycle de 300-900 €.<br><br>**O3 — Fenêtre "souveraineté + open source"** : DINUM, code.gouv.fr, SILL (Socle Interministériel de Logiciels Libres). Faire entrer votre brique au SILL est un label gratuit qui vaut 10 commerciaux.<br><br>**O4 — Financement non-client disponible** : DETR/DSIL pour les communes, Fonds Vert (adaptation climatique / feux de forêt), appels à projets BPI i-Nov, et surtout **Horizon Europe Cluster 3 (Civil Security for Society) / DG ECHO Prevention & Preparedness** : subventions 0,5-2 M€ en consortium, et l'open source y est un bonus de notation.<br><br>**O5 — MCP / agents IA : vous êtes 18 mois en avance.** Exposer les zones et alertes comme outils MCP est aujourd'hui inexistant dans ce secteur. C'est un angle presse et un angle "labo d'innovation" (CREOGN gendarmerie, ENSOSP, Pôle Risques).<br><br>**O6 — International francophone & feux de forêt** : Portugal, Grèce, Espagne, Maghreb, Afrique de l'Ouest, DROM. Le SMS-zone est encore plus pertinent là où la data est chère/absente. | **THREATS**<br><br>**T1 — NexSIS 18-112 / ANSC. La menace structurelle n°1.** Programme d'État porté par l'Agence du Numérique de la Sécurité Civile, destiné à remplacer tous les SGA/SGO des SDIS, avec volet cartographique et mobilité. Tout achat SDIS est aujourd'hui arbitré à l'aune de "est-ce que NexSIS le fera ?". **La réponse d'un DDSIS sera "on attend NexSIS" dans 70 % des cas.** *Donc : ne jamais se positionner en concurrent de NexSIS. Se positionner en "dégradé / hors SGO / hors réseau", le seul terrain que NexSIS ne couvrira jamais bien.*<br><br>**T2 — Esri France / ArcGIS.** Field Maps + Survey123 + Dashboards + Experience Builder. Présent dans une majorité de SDIS via le groupement SIG, marchés déjà passés, UGAP. Fait 80 % de vos cas d'usage, mal, mais déjà acheté et déjà validé par la DSI. *Donc : votre réponse en une phrase doit être prête — "Field Maps ne marche pas sans réseau pour PARTAGER une zone créée sur le terrain ; nous, si, par SMS."*<br><br>**T3 — Systel / Hexagon (I/CAD) / éditeurs SGO installés.** Ils peuvent ajouter un module de zonage mobile en un trimestre. Ils ont les contrats-cadres. *Donc : vitesse + niche + open source comme fossé, pas la techno.*<br><br>**T4 — FR-Alert (cell broadcast, État, gratuit)** couvre l'alerte des populations. Si vous mettez un pied sur ce terrain, vous êtes mort. *Donc : jamais d'alerte population, uniquement de l'alerte intervenants.*<br><br>**T5 — Predict Services** (Météo-France/Airbus/BRGM) sur le segment communes : anticipation inondation, abonnement communal, déjà implanté, financé parfois par les assureurs. Concurrent frontal sur le budget PCS. *Donc : complémentarité (eux = prévision, vous = terrain/zonage/diffusion), pas affrontement.*<br><br>**T6 — Les SDIS développent en interne.** Plusieurs SDIS ont des cellules SIG/dev (13, 33, 31, 59, 69, 78…). L'open source leur permet de **prendre votre code et de ne jamais vous payer**. C'est le risque majeur de l'AGPL mal pensée. *Donc : open core — le cœur libre, mais le serveur multi-tenant, le connecteur SGO, l'admin de flotte, les tuiles pré-packagées et le MCP restent en licence commerciale.*<br><br>**T7 — MDM / PSSI sur téléphones pro.** NEOGEND (gendarmerie) et les terminaux SDIS sous MDM bloquent l'installation PWA et parfois le navigateur hors whitelist. *Donc : le marché initial, c'est le téléphone PERSONNEL du volontaire — assumez-le, c'est aussi votre argument RGPD le plus délicat (voir §8).*<br><br>**T8 — Militaire : fermé.** Zonage de tir/manœuvre = besoin d'homologation SSI (II 901), de confidentialité Défense, de matériel qualifié. Un freelance sans habilitation n'entre pas. *Donc : militaire = horizon 3 ans via un industriel porteur, pas un segment de lancement.*<br><br>**T9 — Responsabilité & assurance.** Pas de RC Pro = un seul incident vous ruine. |

## 1.2 Les 5 actions prioritaires qui en découlent

| # | Action | Délai | Effort | Pourquoi (issue du SWOT) |
|---|---|---|---|---|
| **1** | **Recruter un co-porteur officier sapeur-pompier** (capitaine/commandant actif ou récemment retraité, idéalement chargé SIG ou formateur ENSOSP/CFT). Statut : associé minoritaire ou ambassadeur rémunéré à la commission (10-15 %). | J+30 | Moyen | Neutralise **W9** (absence de légitimité terrain), ouvre **O2**, crédibilise face à **T1**. Sans lui, votre taux de réponse en prospection SDIS sera < 2 %. |
| **2** | **Re-tarifer intégralement** : abonnement annuel par paliers (490 / 1 200 / 3 500 / 9 000 / 25 000 € HT), formation intra à 4 500 € HT, intégration à 650 €/j. | J+10 | Faible | Corrige **W1, W2, W3**. C'est l'action au meilleur ratio impact/effort du plan. |
| **3** | **Pivoter le discours FIRMS** : supprimer "détection d'incendie" de toute la communication, remplacer par "veille satellite et suivi de front à l'échelle départementale, en complément du 18/112". | J+7 | Faible | Corrige **W6**. Un officier feux de forêt démonte ce point en 30 secondes et vous perdez toute crédibilité pour le reste de l'entretien. |
| **4** | **Adopter un modèle open core explicite** : noyau PWA + encodage SMS en AGPL-3.0 ; serveur multi-tenant, connecteurs, console d'administration, serveur MCP et packs de tuiles en licence commerciale. Déposer la marque à l'INPI (~250 €). | J+45 | Moyen | Neutralise **T6** (auto-hébergement gratuit par les SDIS) tout en conservant le bénéfice **S3/O3**. |
| **5** | **Gagner 3 références payantes sur les AASC / CCFF / communes avant tout SDIS.** Objectif : 3 contrats × 500-1 500 € en 90 jours, avec autorisation écrite de citer le nom et une vidéo de témoignage. | J+90 | Moyen | Attaque **O2**, construit la preuve sociale qui débloque **W9** et contourne **T1** (ces acheteurs ne connaissent pas NexSIS et n'attendent rien de l'État). |

---

# 2. BUSINESS MODEL

## 2.1 Business Model Canvas

### ① Segments de clientèle (par ordre de priorité réelle, pas d'envie)

| Prio | Segment | Volume France *(est.)* | Canal d'achat réel | Cycle | Décideur |
|---|---|---|---|---|---|
| **P1** | **AASC (unités locales) + CCFF + équipes cynotechniques + réserves communales de sécurité civile (RCSC)** | 2 500 - 4 000 unités locales (ADPC ~500 antennes, Croix-Rouge ~450 unités locales, FFSS, Croix Blanche, SNSM ~215 stations, CCFF ~300-500 en PACA/Corse/Occitanie, RCSC ~700-900 communes) | **Budget propre d'association** (décision du président/trésorier), subvention communale, mécénat local. **Aucun marché public.** Paiement CB ou virement. | **2 à 6 semaines** | Président d'unité locale / chef de CCFF |
| **P2** | **Communes et EPCI soumis à PCS / PICS** | ~16 000-20 000 communes PCS *(est. post-décret 2022-907)*, dont ~12 000 de moins de 3 500 hab. + ~1 250 EPCI | Bon de commande direct (**< 40 000 € HT = procédure sans publicité ni mise en concurrence**), ligne budgétaire "sécurité civile" ou "informatique", **DETR/DSIL** pour l'investissement, parfois via le prestataire PCS existant | **1 à 4 mois** (calé sur le conseil municipal) | DGS, secrétaire de mairie, adjoint sécurité, maire |
| **P3** | **SDIS** | **101** SDIS/SIS + BSPP + BMPM | Marché public. < 40 k€ HT : gré à gré possible mais service achats frileux. Au-delà : MAPA puis procédure formalisée (seuil fournitures/services collectivités ≈ 221 k€ HT, hors sujet pour vous). Possible via **UGAP** si référencement (lourd) ou en **sous-traitance d'un titulaire existant** (voie rapide) | **6 à 18 mois** | Chaîne : chef de groupement SIC/SIG → chef de groupement opérations → DDSIS → président du CASDIS (élu départemental). Le **DPO** et le **RSSI** ont un droit de veto de fait. |
| **P4** | **Gestionnaires d'espaces & risques privés** : ONF, ASA DFCI, coopératives forestières, parcs naturels, gestionnaires de sites industriels/Seveso, organisateurs d'événements (DPS), stations de ski, fédérations de chasse | 2 000 - 5 000 entités | Achat privé libre, devis simple | **2 à 8 semaines** | Responsable exploitation / QHSE |
| **P5 (horizon 3 ans)** | Gendarmerie, armée, sécurité civile d'État | — | SAILMI / AID / marchés Défense, homologation SSI | 18-36 mois | Hors portée en l'état |

> **Verdict de séquencement** : votre brief dit "marché d'entrée : sapeurs-pompiers (SDIS)". **C'est le pire point d'entrée possible** : cycle le plus long, concurrence institutionnelle la plus forte (NexSIS), exigences de sécurité les plus hautes, et zéro référence de votre côté. Entrez par **P1 et P2**, et laissez les SDIS venir à vous par le bouche-à-oreille des volontaires — qui sont, pour 198 000 d'entre eux, **les mêmes personnes** que les bénévoles de P1.

### ② Proposition de valeur

| Segment | Phrase de valeur (à utiliser telle quelle) |
|---|---|
| P1 AASC/CCFF | *« Découpez un secteur de recherche sur la carte, envoyez-le par SMS à vos 20 bénévoles. Ils l'ouvrent sur n'importe quel téléphone, même sans réseau data, même sans application installée. 0 € pour commencer. »* |
| P2 Communes | *« Votre PCS devient opérationnel sur le terrain : les zones, les points de rassemblement et les secteurs des référents de quartier, diffusés en 30 secondes par SMS, sans serveur, sans compte, sans 4G. »* |
| P3 SDIS | *« Le mode dégradé de votre cartographie opérationnelle. Quand le réseau tombe ou que le SGO n'atteint pas le SPV, une trame de 35 caractères suffit à transmettre un périmètre. Open source, auditable, hébergeable chez vous, réversible par construction. »* |
| P4 Privé | *« Le zonage partagé sans infrastructure, pour vos équipes en zone blanche. »* |

### ③ Canaux
- **Dépôt GitHub + démo publique instantanée** (le canal n°1 : un officier teste à 23 h depuis son canapé).
- **Bouche-à-oreille volontaire** (le seul canal qui convertit vraiment dans ce milieu).
- **Webinaires** (§3).
- **Fédérations & unions départementales** (FNSPF, UDSP, ADPC nationale, ANCCFF).
- **Salons** : Congrès national des sapeurs-pompiers de France, Secours Expo, Salon des Maires et des Collectivités Locales (SMCL), Preventica, GeoDataDays.
- **Prescripteurs** : bureaux d'études PCS, cabinets de conseil en gestion de crise, intégrateurs SIG régionaux.
- **Presse spécialisée** (§3).
- **SILL / code.gouv.fr** (label de confiance gratuit).

### ④ Relation client
- **Self-service total sur le socle libre** (doc, vidéos, Discord/Matrix communautaire).
- **Contrat de support annuel** pour les payants (SLA écrit, pas de Slack ouvert).
- **Communauté de contributeurs "sapeurs-devs"** : l'open source doit produire des contributeurs, sinon il ne produit que des coûts.
- **Un "club utilisateurs" annuel** (visio, 2 h) : c'est le rituel qui fait renouveler les abonnements dans le public.

### ⑤ Flux de revenus
Voir grille détaillée §2.2.

### ⑥ Ressources clés
- Le code (vous).
- **Le co-porteur officier SP** (légitimité — ressource aussi critique que le code).
- Les packs de tuiles pré-générés par département (actif sous-estimé : c'est pénible à faire, ça a de la valeur, et c'est facilement monétisable : 290 € le pack départemental hors-ligne prêt à l'emploi).
- Les références nommées.
- La marque.

### ⑦ Activités clés
Développement · Formation · Intégration · **Animation de communauté** · Réponse aux consultations · Veille réglementaire (loi Matras, Beauvau de la sécurité civile, NexSIS).

### ⑧ Partenaires clés
| Partenaire | Nature | Intérêt pour eux |
|---|---|---|
| Bureaux d'études PCS / PICS | Revente / apport d'affaires (20 %) | Ils livrent un PCS papier, vous le rendez opérationnel → ils facturent plus cher |
| Intégrateurs SIG régionaux | Groupement momentané d'entreprises (GME) pour répondre aux marchés SDIS | Ils ont les références, vous avez la brique |
| ENSOSP / CNFPT / écoles départementales | Intervention en formation | Contenu neuf sur l'IA et le numérique opérationnel |
| UDSP / FNSPF | Référencement, tribune, congrès | Service aux volontaires |
| Éditeur SGO (Systel, Hexagon) | Partenariat techno (export de zone vers leur système) | Fonction qu'ils n'ont pas |
| Opérateur SMS (OVH, Octopush, Brevo) | Revente d'envois SMS | Marge sur volume |
| Mutuelles / assureurs | Mécénat et financement (§4) | Prévention = moins de sinistres |

### ⑨ Structure de coûts (année 1, réaliste)

| Poste | Montant annuel |
|---|---|
| Hébergement (VPS + CDN tuiles + backup) | 1 200 - 2 400 € |
| RC Pro éditeur + cyber | 1 500 - 3 500 € |
| Dépôt marque INPI + CGU/CGV/DPA rédigés par un avocat IT | 2 500 - 4 500 € (one-shot) |
| Salons (stand partagé Congrès SP ou SMCL) | 3 000 - 9 000 € |
| Déplacements / démonstrations | 2 500 € |
| Outils (CRM, emailing, visio, design) | 900 € |
| Qualiopi (optionnel, si axe formation) | 4 000 - 8 000 € (one-shot) + 1 500 €/an |
| **Total cash année 1** | **12 000 - 28 000 €** |
| **Coût d'opportunité (le vrai coût)** | **470 € × ~120 j = 56 400 €** |

> **Le chiffre qui doit piloter toutes vos décisions** : à 470 €/j de TJM, chaque mois à mi-temps sur ce projet coûte **~5 000 € de CA non facturé**. Le projet doit générer **≥ 60 000 € HT en année 1** pour simplement égaler le statu quo. Toute la grille tarifaire ci-dessous est construite pour cet objectif.

## 2.2 Grille tarifaire recommandée

### A. Logiciel / abonnement (récurrent annuel — facturation annuelle d'avance, jamais mensuelle)

| Offre | Cible | Prix HT/an | Contenu |
|---|---|---|---|
| **Libre** | Tous | **0 €** | Code AGPL, auto-hébergement, doc publique, communauté. Aucun support. |
| **Terrain** | AASC, CCFF, RCSC, amicales, assos | **490 €** | Instance hébergée mutualisée, 1 pack de tuiles départemental, 30 utilisateurs, support e-mail 5 j ouvrés, mises à jour |
| **Commune** | Communes < 10 000 hab. | **1 200 €** | + zones PCS pré-chargées, 100 utilisateurs, 500 SMS/an inclus, export PDF du zonage, 1 visio de prise en main (1 h) |
| **Intercommunalité / Grande commune** | EPCI, PICS, communes > 10 000 hab. | **3 500 €** | + multi-communes, comptes administrateurs multiples, 2 000 SMS/an, assistance exercice annuel (1/2 j) |
| **Groupement / Association départementale** | UDSP, fédération départementale AASC, ASA DFCI, parc naturel | **9 000 €** | + instance dédiée, 1 000 utilisateurs, marque blanche légère, support 2 j ouvrés, 2 j de formation incluses |
| **SDIS / Opérateur** | SDIS, grande entreprise, armée | **18 000 - 35 000 €** | + instance dédiée ou on-premise, SLA 4 h ouvrées, connecteur SGO/SIG, serveur MCP, réunion trimestrielle, engagement de réversibilité, assistance à l'homologation |

**Options transverses :**
| Option | Prix |
|---|---|
| Pack tuiles hors-ligne départemental (OSM + IGN + relief, prêt à embarquer) | 290 € HT one-shot / département |
| Crédits SMS | 0,055 € / SMS (marge ~45 %) |
| Hébergement souverain qualifié / on-premise avec assistance | +40 % |
| Connecteur sur mesure (SGO, SIG départemental, Predict, main courante) | 4 500 - 12 000 € forfait |

### B. Formation

| Offre | Format | Prix HT | Commentaire |
|---|---|---|---|
| **Découverte** | Webinaire 1 h | **0 €** | Outil d'acquisition, pas une offre |
| **Prise en main opérationnelle** | 1 j, intra, 6-12 pers. | **2 400 €** | Le produit d'appel payant |
| **Intégration & administration** | 2 j, intra, 6-12 pers. | **4 800 €** | *(Remplace vos 940 € — c'est un ×5, et c'est le prix du marché公 public pour 2 j en intra)* |
| **Inter-entreprises (en ligne)** | 2 × 3 h, 12 pers. max | **590 €/participant** | Scalable, zéro déplacement. 12 × 590 = 7 080 € pour 6 h de travail. **Votre meilleure marge horaire.** |
| **Formation de formateurs** | 3 j + licence pédagogique | **9 500 €** | Vendue aux SDIS/UDSP qui démultiplient en interne. Supprime votre charge de formation. |

### C. Intégration / prestation

| Offre | Prix |
|---|---|
| TJM intégration & développement spécifique | **650 € HT/j** *(pas 470 € — vous n'êtes plus un fullstack générique, vous êtes le seul expert de votre propre brique en milieu sécurité civile)* |
| TJM conseil / AMOA gestion de crise numérique | **850 € HT/j** |
| Forfait "exercice de sécurité civile instrumenté" (préparation + animation + RETEX) | **4 500 € HT** |
| Audit d'architecture / assistance homologation | **3 500 € HT** |

### D. Chemin de conversion (le tunnel)

```
GitHub / démo publique (0 €)
   │  déclencheur : un bénévole teste et l'utilise en exercice
   ▼
Pack tuiles départemental (290 €)  ← 1er paiement, sans friction, sans marché public
   │  déclencheur : "on veut que ça marche pour toute l'unité"
   ▼
Abonnement Terrain (490 €/an) ou Commune (1 200 €/an)
   │  déclencheur : un exercice réussi + un élu qui le voit
   ▼
Formation 1 j (2 400 €) → Formation 2 j (4 800 €)
   │  déclencheur : plusieurs unités d'un même département
   ▼
Offre Groupement départementale (9 000 €/an)
   │  déclencheur : le SDIS voit que 15 CCFF du département l'utilisent déjà
   ▼
SDIS (18-35 k€/an) + intégration (15-40 k€ one-shot)
```


> **Le point clé** : vous ne vendez pas au SDIS, **vous rendez le SDIS incapable de l'ignorer**. Quand 15 unités locales et 40 communes d'un département utilisent l'outil, le chef de groupement opérations ne décide plus d'acheter, il décide de régulariser. C'est le seul mode de vente qui fonctionne en bottom-up dans la sécurité civile française.

## 2.3 Potentiel chiffré du marché

| Segment | Cible adressable | Taux de pénétration visé à 3 ans | Clients | Panier moyen HT/an | CA récurrent |
|---|---|---|---|---|---|
| AASC / CCFF / RCSC | ~3 000 unités | 4 % | 120 | 490 € | 58 800 € |
| Communes PCS | ~16 000 | 0,8 % | 128 | 1 200 € | 153 600 € |
| EPCI / PICS | ~1 250 | 2 % | 25 | 3 500 € | 87 500 € |
| Groupements départementaux | ~101 départements | 8 % | 8 | 9 000 € | 72 000 € |
| SDIS | 101 + BSPP + BMPM | 5 % | 5 | 24 000 € | 120 000 € |
| Privé / gestionnaires | ~3 000 | 1 % | 30 | 1 500 € | 45 000 € |
| **TOTAL récurrent an 3** | | | **316 clients** | | **~537 000 €** |
| **+ Formation & intégration an 3** | | | | | **~120 000 €** |
| **TOTAL an 3** | | | | | **~657 000 € HT** |

**Trajectoire réaliste (scénario médian, 1 personne + 1 co-porteur à temps partiel) :**

| | An 1 | An 2 | An 3 |
|---|---|---|---|
| Clients récurrents | 18 | 85 | 316 |
| CA récurrent | 14 000 € | 78 000 € | 537 000 € |
| CA formation + intégration | 38 000 € | 85 000 € | 120 000 € |
| **CA total** | **52 000 €** | **163 000 €** | **657 000 €** |
| Jours freelance conservés en parallèle | 90 j (42 k€) | 40 j (19 k€) | 0 |

> **An 1 à 52 k€ est en dessous de votre statu quo freelance (~95-110 k€ pour 220 j).** C'est normal et c'est le prix d'entrée. Mais vous devez décider maintenant : **êtes-vous prêt à perdre ~50 k€ de revenu en année 1 ?** Si non, le projet doit rester à 1 j/semaine et le plan 90 jours ci-dessous doit être étalé sur 180.

---

# 3. WEBINAR & VISIBILITÉ — Plan 90 jours

## 3.1 Le webinaire

| Élément | Choix | Justification |
|---|---|---|
| **Titre** | **« Transmettre un périmètre d'intervention par SMS, sans réseau data, sans application à installer »** | Concret, vérifiable, pas de buzzword. Un officier comprend l'intérêt en lisant le titre. |
| **Sous-titre / accroche** | *« 35 caractères suffisent pour envoyer une zone d'évacuation, un secteur de recherche ou une DZ à toute votre équipe — y compris en zone blanche, y compris sur un téléphone à 80 €. Démonstration en direct, outil open source, rien à acheter. »* | Chiffre précis + objection budget levée d'avance + preuve en direct |
| **Format** | 45 min + 15 min Q&R, **mardi ou jeudi 20 h 30** | **Critique** : les SPV et bénévoles AASC travaillent la journée. Un webinaire à 14 h ne touche que les professionnels. Faites-en **deux sessions** : une à 12 h 30 (pros, SDIS, communes) et une à 20 h 30 (volontaires). |
| **Plateforme** | **Livestorm** (RGPD, hébergement UE, inscription simple, replay auto) ou **BigBlueButton auto-hébergé** (cohérence open source, argument de vente en soi). Éviter Zoom US pour ce public. | Crédibilité souveraineté |
| **Audience cible** | Chefs de centre, chefs de groupe, officiers CODIS, référents SIG SDIS, présidents d'unités AASC, chefs de CCFF, DGS et secrétaires de mairie, élus référents sécurité, bureaux d'études PCS | |
| **Objectif chiffré** | 120 inscrits → 55 présents → 20 demandes de démo/pack → **5 ventes (pack tuiles ou abonnement Terrain)** | |

### Plan des 45 minutes (minutage strict)

| Temps | Séquence | Contenu | Règle |
|---|---|---|---|
| **0-3 min** | **Le problème, en situation réelle** | Un récit : feu de broussailles, vallée encaissée, 3 CIS engagés, pas de 4G, le COS doit transmettre un périmètre de sécurité à 4 chefs d'agrès. Comment on fait aujourd'hui ? Radio + description verbale + "tu prends le chemin après le transfo EDF". | **Pas de slide avec votre nom.** On commence par leur monde. |
| **3-6 min** | Qui parle | Vous (30 s) + **le co-porteur officier SP (2 min)** — c'est lui qui donne la légitimité | Le co-porteur ouvre et ferme. Vous êtes la technique. |
| **6-16 min** | **DÉMO 1 — Le SMS-zone** | En direct : tracer une zone, générer la trame, l'envoyer sur un vrai téléphone Android bas de gamme filmé en incrustation, la coller, la zone apparaît. **Puis couper le wifi du téléphone et refaire.** | Le moment décisif de tout le webinaire. Répétez-le 10 fois avant. Prévoyez une vidéo de secours pré-enregistrée. |
| **16-24 min** | **DÉMO 2 — Recherche de personne disparue** | Découper une zone en 6 secteurs, assigner, envoyer 6 SMS différents, suivre la progression du ratissage | Le cas d'usage qui parle le plus aux AASC et à la gendarmerie |
| **24-31 min** | **DÉMO 3 — Alerte automatique sur zone** | Point de chaleur satellite / vigilance Météo-France entrant dans un polygone → notification. **Présenté comme veille, pas comme détection.** | Honnêteté sur la latence : dites-le vous-même avant qu'on vous le demande. Ça vous fera gagner la salle. |
| **31-36 min** | **Hors-ligne, PWA, sécurité, RGPD** | Pas de compte, pas de tracking, pas de store, données qui ne sortent pas, code auditable | Anticipe les objections DSI/DPO |
| **36-41 min** | **Comment l'avoir** | Gratuit et open source maintenant / pack départemental 290 € / abonnement 490-1 200 € / formation. **Un seul appel à l'action : "demandez votre pack départemental de test, gratuit pendant 60 jours."** | Un seul CTA. Pas trois. |
| **41-45 min** | **Appel à contributeurs et à terrains d'essai** | « Je cherche 5 unités pour un exercice grandeur nature ce trimestre. » | Transforme un webinaire en pipeline |
| **45-60 min** | Q&R | | Enregistrez les questions : c'est votre roadmap et votre FAQ |

**Après le webinaire (J+1, automatisé) :** e-mail avec replay + lien démo + PDF 2 pages "trame SMS, mode d'emploi" + lien de réservation d'un créneau de 20 min.

## 3.2 Les 5 canaux complémentaires

| # | Canal | Actions concrètes | Coût | Résultat visé à 90 j |
|---|---|---|---|---|
| **1** | **LinkedIn** | 3 posts/semaine, **format obligatoire : vidéo verticale de 30-60 s montrant une manip réelle**, pas de texte seul. Cibler par commentaire les publications de FNSPF, ANDSIS, SDIS, AMF, AMRF, Préventique. Séquence de 12 posts : le problème (3), la démo (4), le RGPD/souveraineté (2), l'open source (1), les témoignages (2). Contacter nominativement **150 personnes** : chefs de groupement SIC/SIG des 101 SDIS, présidents d'ADPC départementales, DGS de communes à risque. | 0 € | 1 500 abonnés qualifiés, 25 conversations, 8 démos |
| **2** | **Presse spécialisée** | Dossier de presse + vidéo de 90 s envoyés à : *Sapeurs-Pompiers de France* (FNSPF), *Face au Risque*, *Soldats du Feu Magazine*, *Secours Mag*, *Préventique*, *La Gazette des Communes*, *Le Courrier des Maires*, *Techni.Cités*, *InfoProtection*, *Décryptagéo* / *SIGmag* (presse SIG). **Angle à utiliser : « Un développeur français met en open source un outil de zonage qui fonctionne sans réseau »** — angle souveraineté + gratuité + zone blanche. C'est une histoire que ces titres publient. | 0 € (+ 400 € si rédaction par un pigiste) | 2 à 4 parutions |
| **3** | **Fédérations & réseaux institutionnels** | FNSPF (service innovation + rubrique partenaires), **les 96 UDSP** (mail personnalisé au président), ANDSIS, Fédération nationale de protection civile (ADPC), Croix-Rouge (pôle urgence/secourisme), ANCCFF, Association des Maires Ruraux de France (AMRF — très réceptive aux outils gratuits pour petites communes), AMF (groupe de travail sécurité civile), **ENSOSP** (division recherche / Pôle Risques Méditerranée), CREOGN pour l'axe gendarmerie. | 0 € | 2 relais officiels, 1 intervention programmée |
| **4** | **Événements** | **Court terme (J0-J90)** : rencontres départementales et AG d'UDSP/ADPC (gratuites, 50-150 personnes très qualifiées, on vous laisse parler 20 min si vous le demandez). **Secours Expo** (Paris, hiver). **Moyen terme** : Congrès National des Sapeurs-Pompiers de France (l'événement cardinal du secteur, ~50 000 visiteurs, stand partagé ou présence visiteur avec RDV pris d'avance), **Salon des Maires et des Collectivités Locales** (Paris, novembre — le meilleur ROI pour le segment P2), **Preventica**, **GeoDataDays** (AFIGEO, pour le réseau SIG). **Règle : année 1, allez-y en visiteur avec 25 RDV pris à l'avance, pas en exposant.** Un stand à 6 000 € sans référence ne convertit pas. | 800 - 3 000 € | 40 contacts qualifiés |
| **5** | **Open source & communautés techniques** | Dépôt GitHub soigné (README avec GIF de démo en premier écran, en FR **et** EN), publication sur Hacker News / Reddit r/webdev + r/gis + r/sapeurspompiers, **inscription au SILL / code.gouv.fr**, BlueHats, Capitole du Libre / Open Source Experience. Publier un **article technique de fond** : *« Encoder un polygone en 35 caractères pour le transmettre par SMS »* — c'est un contenu qui circule et qui vous fait exister comme auteur de la méthode. | 0 € | 400 ★ GitHub, 3 contributeurs, label SILL en cours |

## 3.3 Calendrier semaine par semaine (90 jours)

| Semaine | Produit / Technique | Marketing / Commercial | Livrable mesurable |
|---|---|---|---|
| **S1** | Démo publique en ligne stable + 3 jeux de données de démonstration (feu, RPS, DPS) | Nouvelle grille tarifaire actée. Page de vente écrite. | URL de démo qui marche sur un Android bas de gamme |
| **S2** | README + GIF + doc d'installation en 5 min | **Liste de 300 prospects nominatifs** constituée (SDIS SIG, UDSP, ADPC, CCFF, communes PCS) | Fichier CRM de 300 lignes |
| **S3** | Export PDF du zonage (livrable indispensable pour le monde public) | **Recrutement du co-porteur officier SP** — 10 contacts, 3 entretiens | Un accord de principe signé |
| **S4** | Checksum + version de la trame SMS (anti-erreur) | Lancement LinkedIn : 3 posts. Dossier de presse envoyé à 10 titres. | 1ᵉʳ post > 10 000 vues |
| **S5** | Pack tuiles hors-ligne : automatisation de la génération par département | Page d'inscription au webinaire en ligne. Invitation à 300 prospects + LinkedIn. | 40 inscrits |
| **S6** | Mode "exercice" (données fictives, bandeau rouge) | Relances. Sollicitation des UDSP/ADPC pour relayer l'invitation. | 90 inscrits |
| **S7** | Durcissement : CGU, mentions, bandeau de responsabilité | Répétition générale du webinaire devant 3 pompiers (impitoyables). Correction. | Script validé |
| **S8** | — (gel du code, zéro déploiement cette semaine) | **WEBINAIRE ×2 (12 h 30 et 20 h 30)**. E-mail de suivi J+1. | 55 présents cumulés, 20 demandes |
| **S9** | Corrections issues des questions du webinaire | **20 entretiens de 20 min** avec les demandeurs. Qualification. | 10 prospects chauds |
| **S10** | Fonction la plus demandée en webinaire (et une seule) | **Première proposition commerciale envoyée.** Ciblage : 2 AASC + 3 communes. | 5 devis envoyés |
| **S11** | Connecteur Météo-France vigilance (facile, visible, utile) | **Premier exercice terrain instrumenté** avec une unité pilote. Photos + vidéo. | 1 exercice réalisé, filmé |
| **S12** | Serveur MCP v0 (angle presse/innovation) | Publication du cas d'usage terrain (LinkedIn + presse + GitHub). Relance des devis. | **2 premières ventes** |
| **S13** | Stabilisation, dette technique | Bilan chiffré. Demande d'intervention à l'AG d'une UDSP. Dépôt SILL. Préparation du webinaire #2. | **3 clients payants, 1 témoignage écrit** |

> **Objectif 90 jours, non négociable : 3 clients payants et 1 vidéo de témoignage.** Sans ça, le modèle n'est pas validé et il faut revoir le segment avant d'investir davantage.

---

# 4. ÉLUS, COLLECTIVITÉS, MUTUELLES & ASSUREURS

## 4.1 Les élus locaux — vos financeurs naturels (et vos meilleurs prescripteurs)

**Pourquoi eux :** le maire est **directeur des opérations de secours (DOS)** sur sa commune (CGCT art. L2211-1 et L2212-2). Il porte une responsabilité personnelle, y compris pénale, en cas de carence dans l'organisation des secours. Et depuis la loi Matras, il a une **obligation légale de PCS** qu'il remplit souvent avec un classeur jamais ouvert. C'est une douleur juridique, pas une envie technologique.

**Angle d'approche :**
1. **Jamais par le service informatique.** Passez par le **DGS** ou la **secrétaire de mairie** (petites communes) ou l'**adjoint à la sécurité**.
2. **Entrez par l'exercice, pas par le logiciel.** Proposez d'animer gratuitement un exercice de sécurité civile de 2 h sur la commune, avec votre outil. Un maire refuse un devis, il ne refuse pas un exercice gratuit qui le met en valeur devant ses administrés et sa presse locale.
3. **Vecteurs** : AMRF (Association des Maires Ruraux de France) et AMF départementales, congrès départementaux des maires (chaque département en organise un, gratuit, 100-400 maires présents), le SMCL.

**Argumentaire en 3 points (à dire mot pour mot) :**
| # | Argument | Formulation |
|---|---|---|
| 1 | **Responsabilité** | *« Votre PCS existe sur le papier. Le jour J, vos référents de quartier sauront-ils où se trouve exactement leur secteur, sans réseau ? Cet outil transforme votre PCS en instructions reçues sur le téléphone de chacun en 30 secondes. »* |
| 2 | **Coût & financement** | *« 1 200 € par an, c'est moins cher qu'une seule réunion publique. Et c'est éligible à la DETR ou à la DSIL en investissement, avec un taux de subvention courant de 30 à 50 %. Je vous fournis la fiche technique pour le dossier. »* |
| 3 | **Inclusion / ruralité** | *« Ça marche sur le vieux téléphone du bénévole de 68 ans, sans application à installer, sans compte à créer, et dans les zones où il n'y a pas de 4G. Aucune autre solution ne le fait. »* |

**Leviers de financement public à documenter dans une fiche d'une page (actif commercial majeur) :**
- **DETR** (Dotation d'Équipement des Territoires Ruraux) — enveloppe préfectorale annuelle, dossiers déposés en **janvier-février**. *Caler votre prospection des communes sur ce calendrier est l'optimisation la plus rentable de tout ce plan.*
- **DSIL** (Dotation de Soutien à l'Investissement Local).
- **Fonds Vert** — volet prévention des risques et adaptation au changement climatique (feux de forêt, inondations).
- Subventions régionales / départementales sur la sécurité civile.
- **FEDER** sur les axes résilience.

## 4.2 Mutuelles, assureurs, fondations

| Financeur | Nature du budget | Angle d'approche | Argumentaire en 3 points |
|---|---|---|---|
| **MAIF / Fondation MAIF** | Fondation dédiée à la **recherche en prévention des risques** et à l'éducation à la sécurité. Finance des projets, pas des logiciels commerciaux. | Déposer un **projet de recherche-action** : *« Efficacité de la transmission de consignes géographiques en zone blanche auprès de bénévoles de sécurité civile »* avec une unité pilote et un labo (ENSOSP, université, IMT Mines Alès). Budget demandé : 25-60 k€. | 1) Projet open source = bien commun, aucun enrichissement privé sur la subvention. 2) Public bénévole, pas professionnel = cœur de mission mutualiste. 3) Résultat publiable et réutilisable par tous. |
| **Fondation Macif** | Innovation sociale, économie sociale et solidaire, **utilité sociale locale**. | Monter le dossier avec une **association** (ADPC départementale, CCFF) comme porteur, vous en prestataire. C'est la structure que ces fondations savent financer. | 1) Bénévolat et lien social. 2) Territoires ruraux délaissés. 3) Accessibilité (fonctionne sur matériel ancien, pas de fracture numérique). |
| **Groupama / Fondation Groupama** | **L'assureur du monde rural et agricole.** Assure les communes rurales, les exploitations, les forêts. Très actif en mécénat local via les caisses régionales (13 caisses régionales, chacune avec un budget propre). | **Ne visez pas le siège, visez la caisse régionale.** Angle : protection des forêts et des exploitations agricoles contre l'incendie, outil pour les CCFF et les communes rurales assurées par eux. | 1) Un départ de feu détecté et cadré plus tôt = sinistralité réduite sur leur portefeuille. 2) Visibilité locale auprès des maires (leurs clients). 3) Coût de mécénat dérisoire (3-10 k€) pour un équipement départemental entier. |
| **AXA / Generali / Allianz — fondations prévention** | Fonds prévention des risques naturels et climatiques. | Plus difficile d'accès en tant qu'indépendant. Passer par un consortium. | — |
| **Mutuelles des sapeurs-pompiers & structures de la FNSPF** | Budget d'action sociale et de services aux adhérents. Faible capacité d'investissement technologique. | **Ne les visez pas comme financeurs.** Visez-les comme **canal de diffusion** : une mention dans leur magazine ou leur newsletter vaut plus que leur chèque. | — |
| **UDSP / Amicales de CIS** | Budget propre associatif, de quelques centaines à quelques milliers d'euros, décidé en bureau. | **C'est le vrai acheteur du premier euro.** L'amicale d'un CIS peut décider en 15 jours d'acheter un abonnement à 490 € pour son centre. | 1) Service concret aux volontaires du centre. 2) Prix d'un repas de l'amicale. 3) Valorise le centre vis-à-vis du groupement. |
| **Caisses de dépôt / Banque des Territoires** | Financement de projets de résilience territoriale. | Pertinent uniquement à partir d'un projet départemental ou régional (> 100 k€). | Horizon an 2-3. |

**La règle qui gouverne tout ce chapitre :** une fondation ou une mutuelle ne finance **jamais** une licence logicielle. Elle finance **un projet, sur un territoire, avec une association**. Votre dossier doit donc toujours être : *association porteuse + territoire identifié + objectif mesurable + vous en prestataire*. Préparez **un seul dossier type de 6 pages** et déclinez-le. Budget de temps : 3 jours de rédaction, puis 2 h par dépôt.

---

# 5. CONCURRENCE

## 5.1 Cartographie concurrentielle

| Acteur | Type | Forces | Faiblesses | Positionnement prix *(est.)* | **Votre différenciant** |
|---|---|---|---|---|---|
| **NexSIS 18-112 (ANSC / État)** | SGA-SGO national, déploiement progressif sur les SDIS | Imposé institutionnellement, financé, intégré au 112, légitimité totale | Calendrier long et glissant, périmètre centré salle opérationnelle, pas d'usage en mode déconnecté par le volontaire sur son téléphone perso, rien pour les communes ni les AASC | Financé par l'État / contribution des SDIS | **Ne JAMAIS se positionner contre.** Vous êtes le **mode dégradé** et le **hors-périmètre** (communes, bénévoles, associations). Phrase à utiliser : *« NexSIS est votre système de gestion opérationnelle. Nous, nous sommes ce qui reste quand il n'y a plus de réseau et que la personne à équiper n'est pas dans votre SI. »* |
| **Esri France (ArcGIS Online, Field Maps, Survey123, Dashboards)** | SIG généraliste, standard de fait dans les SDIS et les collectivités | Écosystème complet, déjà acheté, déjà maîtrisé par les cellules SIG, marchés UGAP, support corporate | Lourd, nécessite des comptes nommés, licences coûteuses par utilisateur, **inutilisable par un bénévole occasionnel**, cache hors-ligne mais **pas de partage de zone sans réseau**, dépendance à un éditeur américain (angle souveraineté) | ~2 500 - 10 000 €/an par organisation + licences nommées (de 100 à 600 €/user/an) | **Le partage hors-réseau.** Field Maps consulte hors-ligne, mais ne permet pas de **transmettre** une zone créée sur le terrain à quelqu'un qui n'a pas de data. Votre SMS-zone, si. + Pas de compte, pas de licence par utilisateur, pas de cloud US. |
| **QGIS + plugins** | SIG libre | Gratuit, puissant, utilisé par les services techniques | Desktop, pas mobile, pas temps réel, inutilisable en intervention | 0 € | Vous êtes le mobile terrain que QGIS ne sera jamais. **Positionnez-vous en complément : import/export GeoJSON natif.** |
| **Systel, Hexagon/Intergraph (I/CAD), autres éditeurs SGO** | Systèmes de gestion opérationnelle historiques des SDIS | Contrats en place, intégration CTA/CODIS, connaissance métier | Fermés, chers, lents à évoluer, pas de doctrine "téléphone personnel du volontaire" | Contrats pluriannuels à 6-7 chiffres | **Partenariat plutôt qu'affrontement** : proposez un export de zone depuis leur système vers votre trame SMS. Vous devenez une fonctionnalité qu'ils n'ont pas, sans investissement de leur part. |
| **Predict Services** (Météo-France / Airbus DS / BRGM) | Anticipation et accompagnement des communes et entreprises face aux risques météo/inondation | Très implanté sur le segment communes, caution scientifique, financé parfois par les assureurs, astreinte 24/7 humaine | Prévision et alerte, **pas de zonage tactique collaboratif**, pas de découpage en secteurs, dépend du réseau | ~800 - 5 000 €/an par commune | **Complémentarité frontale assumée** : eux anticipent et alertent, vous organisez le terrain. Cherchez le partenariat, pas le duel : leurs clients communes sont exactement vos prospects. |
| **FR-Alert (État)** | Alerte des populations par diffusion cellulaire | Gratuit, universel, pas d'app à installer | Sens unique, pas de carte précise, réservé aux autorités, **alerte population uniquement** | 0 € | Périmètre totalement disjoint. **Ne jamais empiéter.** Mentionnez-le vous-même pour montrer que vous connaissez le paysage. |
| **Gedicom, Cedralis, Everbridge, Mon Alerte / automates d'appel communaux** | Alerte téléphonique de masse des habitants | Simples, connus des mairies, souvent déjà achetés | Pas de cartographie, pas de terrain, pas de zonage | 500 - 3 000 €/an | Vous êtes la couche **carte et intervenants**, ils sont la couche **population**. Partenariat possible. |
| **Outils de crise : Crisotech, Mémoris, Resolution/Crisis Suite, Astreinte+, "crisis rooms"** | Gestion de cellule de crise (main courante, annuaires, procédures) | Structurés, conformes aux attentes des grandes organisations | Orientés salle de crise et traçabilité, **pas terrain**, pas hors-ligne, interfaces lourdes | 5 000 - 50 000 €/an | Terrain vs salle. Intégration possible via MCP/API. |
| **Intterra, Tablet Command, Technosylva, Pano AI, Watch Duty** (US/Espagne) | Feux de forêt : commandement mobile, modélisation, détection, alerte citoyenne | Très matures sur le feu de forêt, financés | **Quasi absents du marché français**, anglophones, cloud US (rédhibitoire RGPD/souveraineté pour un SDIS), chers | 20 000 - 200 000 €/an | Souveraineté + prix + hors-ligne + langue. Mais surveillez-les : une entrée en France via un partenaire est possible à 24 mois. |
| **Waze / Google Maps / WhatsApp + capture d'écran** | **Le vrai concurrent quotidien** | Gratuit, installé partout, connu de tous | Imprécis, pas de polygone partageable, pas de secteur, pas de traçabilité, données chez Meta/Google (problème RGPD majeur que personne ne regarde) | 0 € | **C'est contre ça que vous vous battez réellement, pas contre Esri.** Aujourd'hui un chef de groupe envoie une capture d'écran annotée sur WhatsApp. Votre argument : *« Une capture d'écran ne se superpose pas à la carte du destinataire et ne lui dit pas s'il est dedans ou dehors. »* |
| **Développement interne SDIS** (cellules SIG de SDIS 13, 33, 31, 59, 69, 78…) | Développement maison | Gratuit pour eux, adapté, maîtrisé | Non maintenu, non documenté, part avec l'agent qui l'a écrit | Coût agent | **Votre open source est l'arme et la faille.** Ils peuvent forker. Contre-mesure : open core + offre "support de votre instance auto-hébergée" à 6 000 €/an — vous vendez la pérennité, pas le code. |

## 5.2 Le risque institutionnel — à traiter frontalement

**Scénario de mort n°1 :** l'ANSC, la DGSCGC ou un SDIS pilote intègre une fonction de zonage partagé dans NexSIS, ou l'État publie un outil équivalent sous licence libre. Votre marché SDIS disparaît en 6 mois.

**Probabilité à 3 ans : 25-35 %** *(est.)* — modérée, car le SMS-zone est une solution de contournement de panne, et l'État conçoit rarement pour le mode dégradé non maîtrisé.

**Les 4 contre-mesures, par ordre d'efficacité :**

1. **Ne pas construire le business sur les SDIS.** Si 70 % de votre CA vient des communes, AASC, CCFF et du privé, NexSIS ne vous touche pas. **C'est la seule contre-mesure vraiment efficace.**
2. **Devenir la brique que l'institution adopte plutôt que réécrit.** Inscription au SILL, licence permissive sur le format de trame (publiez la **spécification** du format en domaine public — si votre encodage devient un standard de fait, on vous appelle pour l'implémenter).
3. **Open core strict.** Le cœur est libre (ce qui le rend adoptable), le reste est commercial (ce qui le rend monétisable). Ne livrez jamais le multi-tenant et la console d'administration en libre.
4. **Vitesse d'itération.** Un acteur institutionnel livre en 24 mois. Vous livrez en 2 semaines. Votre seul avantage structurel est le temps de cycle — ne le gaspillez pas en construisant du 10 Hz.

---

# 6. ADOPTION DE LA PWA

## 6.1 Freins, par ordre de gravité

| # | Frein | Gravité | Réalité |
|---|---|---|---|
| **F1** | **« Ce n'est pas sur le store, donc ce n'est pas une vraie app. »** | **Élevée** | Culture utilisateur. Un SPV de 55 ans n'installe rien qui ne vienne pas du Play Store. Le terme "PWA" ne doit jamais apparaître devant un utilisateur final. |
| **F2** | **MDM / PSSI sur téléphones professionnels** | **Critique sur gendarmerie et SDIS équipés** | Terminaux NEOGEND sous administration ministérielle, navigateur restreint, ajout à l'écran d'accueil désactivé, pas de service worker tiers autorisé. **Sur ces parcs, votre PWA n'entrera pas sans une validation ministérielle de 12-24 mois.** |
| **F3** | **iOS : la PWA reste un citoyen de seconde zone** | **Élevée** | Installation uniquement via Safari → « Partager » → « Sur l'écran d'accueil » (parcours non découvrable, ~40 % d'abandon). Notifications web disponibles depuis iOS 16.4 **mais uniquement si la PWA est installée**. Éviction du cache si l'app n'est pas ouverte pendant quelques semaines. Pas de géolocalisation en arrière-plan. **Testez systématiquement sur un vrai iPhone, pas sur un simulateur.** |
| **F4** | **Géolocalisation uniquement au premier plan** | **Moyenne** | Vous l'avez identifié. Conséquence opérationnelle : l'écran doit rester allumé → **batterie**. Un téléphone bas de gamme avec l'écran allumé et le GPS actif tient 2 à 4 h. Une recherche de personne dure 8 h. **C'est un vrai problème terrain qui tuera l'usage si vous ne le traitez pas.** |
| **F5** | **Méfiance sur le stockage des données** | **Moyenne** | « Où vont mes positions ? » — question systématique, posée par les représentants du personnel et les syndicats. |
| **F6** | **Absence de garantie de service** | **Moyenne** | Une collectivité veut un contrat, un SLA, un interlocuteur, une RC Pro. L'open source gratuit inquiète autant qu'il rassure. |
| **F7** | **Capacité du matériel** | **Faible à moyenne** | Android entrée de gamme (2-3 Go de RAM) : Leaflet + tuiles + Turf, c'est tenable, mais le cache de tuiles hors-ligne peut consommer 200 Mo à 2 Go. Sur un téléphone à 16 Go déjà saturé de photos, ça coince. |

## 6.2 Leviers

| Levier | Mise en œuvre |
|---|---|
| **Le lien qui marche tout de suite** | Le prospect clique, la carte s'affiche, la zone du SMS se décode. **Zéro installation, zéro compte.** C'est votre atout : ne le gâchez pas avec une bannière d'installation agressive dès la première seconde. |
| **Le QR code** | Affiché au mur du CIS, sur la fiche de garde, en bas d'un ordre d'opération. Le vecteur de distribution le plus naturel dans ce milieu. |
| **Le champion local** | Un seul officier ou chef de centre qui l'adopte équipe 40 personnes en une semaine. Identifiez-le, servez-le, donnez-lui du crédit publiquement. |
| **L'exercice** | Les pompiers n'adoptent pas par démonstration, ils adoptent par **manœuvre**. Un outil utilisé dans un exercice départemental est adopté. Offrez l'animation d'exercices. |
| **Le vocabulaire** | Dites « l'outil », « la carte », « le lien ». Jamais « la PWA », jamais « l'application web progressive ». |

## 6.3 Palliatifs recommandés

| # | Palliatif | Priorité | Détail |
|---|---|---|---|
| **P1** | **Mode démo public permanent, sans inscription** | **Immédiate** | URL avec un scénario pré-chargé (feu, RPS, DPS). C'est votre meilleur commercial. Accessible depuis le README, LinkedIn, le QR code, la signature mail. |
| **P2** | **Export PDF et export image du zonage** | **Immédiate** | **Non négociable dans le secteur public.** Le PDF est la preuve, l'archive, la pièce jointe au RETEX, l'annexe du PCS. Format : carte + coordonnées des sommets + trame SMS imprimée + horodatage + mention « document d'aide, non opposable ». |
| **P3** | **Traitement explicite de la batterie** | **Haute** | Mode « économie » : fréquence GPS réduite à 1 point / 30 s, écran en noir et blanc, désactivation des animations. Et **documentez la consommation réelle** (ex. : « 9 %/h en mode économie sur Samsung A14 ») — ce chiffre dans votre doc rassure plus que n'importe quel argument. |
| **P4** | **Fiche "mode d'emploi" A5 imprimable, recto-verso** | **Haute** | Le milieu fonctionne au papier plastifié dans la poche de la veste. Ça coûte 2 h de travail et ça change le taux d'adoption. |
| **P5** | **Parcours d'installation iOS guidé** | **Haute** | Détection de Safari/iOS → visuel animé montrant « Partager → Sur l'écran d'accueil ». Sans ça vous perdez la moitié des utilisateurs iPhone. |
| **P6** | **Wrapper natif en veille (Capacitor / Tauri Mobile)** | **Moyenne — à déclencher sur condition** | Ne le faites PAS maintenant. **Déclencheurs qui justifient de le faire :** (a) un marché public exige une présence au store, (b) un client exige la géolocalisation en arrière-plan, (c) un parc MDM veut distribuer l'app en interne. Comptez 15-25 jours de travail + comptes développeur (99 $/an Apple, 25 $ Google) + processus de validation. **Préparez l'argumentaire, pas le code.** |
| **P7** | **Distribution par MDM en APK signé** | **Moyenne** | Pour les SDIS équipés de terminaux pro : un APK (wrapper WebView) poussé par leur MDM contourne complètement le débat PWA. C'est souvent plus simple à faire accepter que la PWA elle-même. |
| **P8** | **Page "Sécurité & données" publique** | **Haute** | Une page claire : ce qui est stocké, où, combien de temps, qui y accède, comment effacer. À envoyer au DPO avant même qu'il la demande. Gain de 3 semaines dans chaque cycle de vente publique. |

---

# 7. PERFORMANCE & ARCHITECTURE TEMPS RÉEL

## 7.1 Avertissement préalable : dimensionnez le vrai besoin

Avant toute architecture, confrontez l'exigence aux cas d'usage :

| Cas d'usage | Nombre d'entités mobiles | Fréquence utile | Débit réel |
|---|---|---|---|
| Feu de forêt, dispositif départemental important | 150-400 intervenants | 1 position / 10-30 s | **5 à 40 msg/s** |
| Recherche de personne disparue | 20-120 bénévoles | 1 / 15 s | **1 à 8 msg/s** |
| DPS grand événement | 50-200 secouristes | 1 / 20 s | **2 à 10 msg/s** |
| NASA FIRMS, France entière | quelques centaines de points / jour | 1 lot / 3 h | **négligeable** |
| Vigilances Météo-France | ~100 entités | 1 / h | **négligeable** |

**Conclusion : le besoin réel plafonne autour de 50 messages/seconde, pas plusieurs milliers de points à 10 Hz.**

Un process Node.js seul traite cela avec **1 à 3 % de CPU**. Le 10 Hz n'existe que dans deux scénarios : un SaaS multi-tenant agrégeant 100 départements simultanément (vous n'y serez pas avant 5 ans), ou du suivi de drones/véhicules à haute cadence (ce n'est pas votre offre).

> **Recommandation n°1, et c'est la plus importante de cette section : ne construisez pas l'architecture 10 Hz. Construisez l'architecture 50 msg/s, et concevez-la pour qu'elle puisse grandir. Vous économisez 3 à 5 semaines que vous devez investir dans l'export PDF, le mode exercice et la prospection.**

## 7.2 Architecture cible (progressive, en 3 paliers)

### Palier 1 — « ça suffit » : 0 à 200 msg/s, ≤ 500 clients connectés
**À construire maintenant. Coût : 1 VPS à 20 €/mois.**

```
Sources (GPS clients, FIRMS, Météo-France)
        │  HTTP POST groupé (batch 1-5 s) + WebSocket
        ▼
┌──────────────────────────────────────────────┐
│  Node.js — process unique                    │
│  ├─ Ingestion : file en mémoire, coalescing  │
│  │   (on ne garde que la DERNIÈRE position   │
│  │    par entité par tick)                   │
│  ├─ Tick 4 Hz (pas 10 Hz)                    │
│  ├─ Index spatial : Flatbush / RBush         │
│  │   sur les bbox des zones (rebuild on      │
│  │   change, pas à chaque tick)              │
│  ├─ Pré-filtre bbox → Turf booleanPointIn-   │
│  │   Polygon uniquement sur les candidats    │
│  ├─ Machine à états par (entité, zone) :     │
│  │   DEHORS → DEDANS = événement d'entrée    │
│  │   + hystérésis 20 m + confirmation 2 ticks│
│  └─ Diffusion : par viewport + par zone      │
└──────────────────────────────────────────────┘
        │
   ws (uWebSockets.js) ──► clients
   SSE ──► écrans de supervision passifs
```


**Décisions techniques clés du palier 1 :**
| Sujet | Choix | Justification |
|---|---|---|
| **Tick** | **4 Hz (250 ms)**, pas 10 Hz | L'œil humain et le besoin opérationnel ne distinguent rien au-delà. Divise la charge par 2,5 immédiatement. |
| **Coalescing** | Obligatoire : 1 position par entité par tick, les intermédiaires sont jetées | Rend le coût indépendant de la fréquence d'émission des clients. **C'est l'optimisation la plus rentable du système.** |
| **Index spatial** | **Flatbush** (statique, ultra-rapide, parfait pour des zones qui changent peu) avec repli RBush si mutations fréquentes | 10 000 zones indexées : requête en < 10 µs. Ne partez pas sur PostGIS pour ça. |
| **Géométrie** | Pré-calculer bbox + aire + enveloppe convexe de chaque zone au moment de la création, jamais à chaque tick | Turf est lent si on lui redonne du GeoJSON brut en boucle |
| **Anti-rebond** | Hystérésis spatiale (20-30 m) + confirmation sur 2 ticks consécutifs + anti-spam par entité/zone (1 alerte / 5 min) | **Sans ça, un agent qui marche sur la limite d'une zone génère 40 alertes et tout le monde désinstalle.** Problème produit, pas technique. |
| **Transport** | **WebSocket** pour le bidirectionnel (positions montantes + mises à jour descendantes). **SSE** pour les écrans de supervision en lecture seule (plus simple, reconnexion automatique native, traverse mieux les proxys d'entreprise). | Les deux, selon l'usage. Le SSE est sous-estimé pour les écrans CODIS derrière un proxy. |
| **Backpressure** | Surveiller `socket.bufferedAmount` (ou `getBufferedAmount()` en uWS). Au-delà de 256 Ko : passer le client en **mode dégradé** (fréquence divisée par 4, envoi d'instantanés au lieu de deltas). Au-delà de 1 Mo : déconnexion propre avec code applicatif. | **Ne jamais mettre en file d'attente sans limite.** C'est la cause n°1 de fuite mémoire sur ce type de serveur. |
| **Format de transport** | JSON + compression `permessage-deflate` au palier 1. Deltas encodés (varint / quantification à 1e-5 degré) dès que ça dépasse. | Ne faites pas de binaire tout de suite. |
| **Filtrage** | Envoi limité au **viewport** du client (que vous avez déjà prévu — bonne décision) **+ abonnement par zone** | Réduit le fan-out de 80-95 % |
| **Worker** | **Un seul worker Node** suffit. Utiliser `worker_threads` + `SharedArrayBuffer` pour les positions **seulement si** le CPU du thread principal dépasse 60 %. | |

### Palier 2 — 200 à 2 000 msg/s, 500 à 5 000 clients
**Déclencheurs :** CPU du process > 60 % en pointe · latence p99 > 300 ms · besoin de plus d'une instance (haute disponibilité) · un seul client dépasse 500 utilisateurs simultanés.

| Ajout | Techno | Rôle |
|---|---|---|
| Bus d'événements | **Redis Streams** ou **NATS JetStream** | Découple l'ingestion du calcul ; permet plusieurs instances WebSocket |
| État partagé | **Redis** (dernière position par entité, état dedans/dehors, anti-spam) | Permet de scaler horizontalement les passerelles WS |
| Partitionnement | **Sharding géographique par département ou par cellule H3/geohash** | Un worker par zone géographique. Scalabilité naturelle et alignée sur le métier. |
| Passerelle WS | 2-4 instances `uWebSockets.js` derrière un load balancer avec sessions persistantes | HA |
| Persistance | **PostgreSQL + PostGIS** pour les zones, l'historique, le RETEX (jamais dans le chemin temps réel) | |
| Transport | Passage au binaire (MessagePack ou protobuf) + deltas quantifiés | Divise la bande passante par 3-5 |

### Palier 3 — au-delà
**Déclencheurs :** > 2 000 msg/s soutenus · > 10 000 clients · multi-départements simultanés en saison de feux.

- Réécriture du moteur de matching en **Rust ou Go** (gain réaliste ×10-30 sur le calcul géométrique pur).
- **TimescaleDB** ou **ClickHouse** pour les séries temporelles de positions.
- Indexation **H3** pré-calculée : chaque zone est décomposée en cellules H3, le matching devient une recherche de hash en O(1) au lieu d'un test point-dans-polygone. **C'est le changement de paradigme qui débloque le vrai volume.**
- CDN pour les tuiles, séparé de l'application.

> **Honnêtement : vous n'atteindrez jamais le palier 3. Concevez le palier 1 proprement, documentez les paliers 2 et 3 dans un fichier `ARCHITECTURE.md` — ce document est un argument commercial face à un RSSI, bien plus utile que le code correspondant.**

## 7.3 Métriques à instrumenter dès le jour 1

| Catégorie | Métrique | Seuil d'alerte |
|---|---|---|
| **Latence** | Latence bout en bout p50 / p95 / **p99** (émission client → réception destinataire) | p99 > 500 ms |
| **Latence** | Durée du tick de matching (p99) | > 120 ms sur un tick de 250 ms |
| **Débit** | Messages ingérés/s · événements d'entrée-sortie émis/s · messages diffusés/s | — |
| **Retard** | **Lag de la file d'ingestion** (nb de messages en attente) — *la métrique la plus prédictive d'un effondrement* | > 2 000 |
| **Backpressure** | `bufferedAmount` max et médian par socket · nb de clients en mode dégradé · nb de déconnexions pour saturation | > 5 % des clients dégradés |
| **Connexions** | WS simultanés · taux de reconnexion / min · durée médiane de session | Reconnexions > 10 %/min = problème réseau ou proxy |
| **Système** | CPU par worker · RSS · **pauses GC p99** · event loop lag (`perf_hooks.monitorEventLoopDelay`) | Event loop lag > 50 ms |
| **Métier** | Alertes émises/h · **taux d'alertes annulées en < 10 s (= faux positifs de bordure)** · zones actives · % de clients hors-ligne | Faux positifs > 3 % |
| **Qualité produit** | Taux de décodage réussi des trames SMS · nb de trames rejetées par checksum | Échec > 0,5 % |

**Outillage :** `prom-client` → Prometheus → Grafana, et **OpenTelemetry** dès le début pour le traçage distribué. Coût : 1 jour. Argument de vente face à une DSI : considérable.

---

# 8. CNIL / RGPD

## 8.1 Qualification juridique préalable (à écrire noir sur blanc)

| Question | Réponse pour votre projet |
|---|---|
| **Qui est responsable de traitement ?** | **Le SDIS, la commune ou l'association utilisatrice.** Pas vous. |
| **Quel est votre rôle ?** | **Sous-traitant (art. 28 RGPD)** dès que vous hébergez. **Aucun rôle** si le client auto-héberge le logiciel libre (vous n'êtes alors que fournisseur d'outil). **Cette distinction doit être explicite dans votre documentation — c'est un argument de vente autant qu'une protection.** |
| **Contrat obligatoire ?** | **Oui : un DPA (accord de sous-traitance) signé avant toute mise en production hébergée.** Sans DPA, un DPO de collectivité bloque le projet. |
| **Hébergement** | UE obligatoire. Privilégier un hébergeur français (OVH, Scaleway, Outscale). Pour les SDIS sensibles : viser à terme **SecNumCloud**, mais ne pas le promettre. HDS non requis (pas de données de santé… sauf si vous touchez aux bilans victimes — **ne le faites pas**). |

## 8.2 Checklist de conformité

### A. Bases légales par traitement

| Traitement | Base légale | Référence |
|---|---|---|
| Géolocalisation des intervenants pendant une opération | **Mission d'intérêt public** (art. 6.1.e RGPD), adossée aux missions de sécurité civile du SDIS (CGCT art. L1424-2) ou aux pouvoirs de police du maire (CGCT art. L2212-2) | Pour un SDIS/commune |
| Géolocalisation de bénévoles d'association | **Intérêt légitime** (art. 6.1.f) **+ consentement explicite et révocable à chaque session** | Pour une AASC. Le consentement d'un salarié/agent est juridiquement fragile (lien de subordination) → d'où l'intérêt public ci-dessus pour les SP. **Ne mélangez pas les deux.** |
| Partage d'un zonage géographique (polygones) | **Pas de donnée personnelle** si aucune personne n'est identifiable → hors RGPD | **À documenter : c'est votre meilleur atout de conformité.** |
| Données des personnes recherchées (RPS) | **Mission d'intérêt public**, sous la responsabilité de l'autorité (gendarmerie/police/préfet) | **Ne stockez AUCUNE donnée nominative de personne recherchée.** Secteur = polygone + numéro, rien d'autre. |
| Comptes utilisateurs / annuaire des intervenants | **Exécution de mission d'intérêt public** ou **contrat** | Minimisation : identifiant fonctionnel (« CG1 », « VSAV 23 ») plutôt que nom et prénom |
| Envoi d'alertes (SMS, WhatsApp, e-mail, push) | **Mission d'intérêt public** ou **consentement** selon le contexte | **Attention WhatsApp Business Cloud = Meta = transfert hors UE.** Voir ci-dessous. |
| Logs techniques | **Intérêt légitime** (sécurité du système) | 6 à 12 mois max |

### B. Durées de conservation (à inscrire dans la configuration par défaut, pas seulement dans la doc)

| Donnée | Durée recommandée | Justification |
|---|---|---|
| **Positions GPS en direct** | **En mémoire vive uniquement, purge immédiate en fin d'opération** | La doctrine CNIL sur la géolocalisation des véhicules/salariés est très restrictive (traditionnellement **2 mois maximum**, et seulement si strictement nécessaire). Le plus sûr est de **ne pas persister du tout**. |
| **Traces GPS archivées pour RETEX** | **30 jours maximum, puis anonymisation ou suppression automatique**. Si conservation plus longue : **agrégation sans identifiant** (densité de couverture par secteur, pas « qui est passé où ») | La valeur du RETEX est dans la couverture du terrain, pas dans l'individu. **Le paramètre par défaut doit être : pas d'archivage nominatif.** |
| **Zones / polygones** | Durée du dossier + archivage (pas de DCP) | |
| **Journaux d'alertes (qui a reçu quoi)** | 12 mois | Traçabilité opérationnelle |
| **Comptes utilisateurs** | Suppression 3 mois après la fin de l'engagement | |
| **Logs techniques / sécurité** | 6 mois (12 mois max) | |

### C. Documentation à produire (la « liasse conformité » — 4 jours de travail, débloque tous les cycles de vente publique)

| # | Document | Destinataire |
|---|---|---|
| 1 | **Fiche de registre des traitements pré-remplie** (modèle CNIL) que le client n'a plus qu'à adapter | DPO client — **gain de 3 semaines dans le cycle de vente, c'est le document le plus rentable de tout ce plan** |
| 2 | **DPA (accord de sous-traitance art. 28)** | Service juridique client |
| 3 | **Trame d'AIPD / DPIA** pré-remplie | DPO client. Une AIPD est **probablement requise** : géolocalisation + suivi de personnes physiques + déploiement à grande échelle cochent plusieurs critères de la liste CNIL. **Anticipez-la, ne la subissez pas.** |
| 4 | **Politique de confidentialité et notice d'information** des intervenants (art. 13) | Utilisateurs |
| 5 | **Page « Sécurité & données »** publique | Tout le monde |
| 6 | **Procédure de violation de données** (notification sous 72 h) | Interne + client |
| 7 | **Liste des sous-traitants ultérieurs** (hébergeur, SMS, push) avec localisation | DPO client |

### D. Points spécifiques à votre architecture

| Point | Analyse | Action |
|---|---|---|
| **Mode hors-ligne** | Les données restent sur le terminal (IndexedDB / Cache API). **Le terminal est souvent personnel.** Risque : des traces de secteurs d'intervention persistent sur le téléphone perso d'un volontaire, hors de tout contrôle de l'employeur. | **Bouton « Effacer toutes les données locales » visible en permanence** + purge automatique des données opérationnelles après 7 jours + chiffrement des données sensibles en local (clé dérivée d'un code de session). À documenter explicitement. |
| **Terminaux personnels (BYOD)** | Un agent public utilisant son téléphone personnel pour une mission de service = sujet RH et juridique autant que RGPD. | **Ne jamais exiger le téléphone personnel.** Position à tenir : *« L'outil fonctionne sur le terminal que l'organisation décide d'utiliser. »* Laissez le client porter cette décision. |
| **WhatsApp Business Cloud** | Meta Platforms Ireland + transferts hors UE. Pour un SDIS ou une préfecture, **c'est très probablement rédhibitoire**, et pour le secret des opérations c'est discutable. | **Faites-en une option désactivée par défaut**, avec un avertissement explicite. Mettez en avant SMS (opérateur français), e-mail (serveur du client) et push PWA (votre serveur UE). **Ne mettez pas WhatsApp en avant commercialement.** |
| **Push PWA** | Les endpoints passent par FCM (Google) et APNs (Apple). Le contenu est chiffré de bout en bout (VAPID + chiffrement de charge utile), mais **les métadonnées transitent par des acteurs US**. | **Ne jamais mettre d'information opérationnelle dans le corps de la notification.** Notification générique : *« Nouvelle alerte — ouvrir l'outil »*. Le détail est récupéré par l'app. **C'est de la minimisation par conception, et ça se dit très bien à un DPO.** |
| **Tuiles OSM** | Si vous servez les tuiles depuis un tiers, les adresses IP des utilisateurs et leurs zones d'intérêt fuitent. | **Tuiles auto-hébergées ou pré-téléchargées.** Vous le faites déjà — valorisez-le explicitement comme une mesure de conformité, pas seulement comme une mesure technique. |
| **Serveur MCP** | **Point de vigilance majeur et sous-estimé.** Exposer des zones et alertes à un agent IA signifie potentiellement envoyer des données opérationnelles à un LLM hébergé hors UE. | **Par défaut : MCP en lecture seule, sur données non personnelles (zones uniquement), et avec une liste blanche de modèles.** Documentez : *« Aucune donnée personnelle n'est exposée via MCP. »* Si un client veut plus, c'est à lui d'en porter l'analyse d'impact. |

### E. Les 3 interdits absolus à documenter dans le README

> À placer **en haut du README**, dans un encadré, en français et en anglais. Ce n'est pas de la décoration : c'est votre protection juridique, votre positionnement éthique, et un argument de vente face aux DPO et aux représentants du personnel.

```markdown
## ⛔ Usages formellement exclus

Ce logiciel est conçu pour la coordination géographique d'opérations de secours.
Les usages suivants sont explicitement hors périmètre, non supportés, et
contraires à la licence et à l'esprit du projet.

### 1. AUCUNE SURVEILLANCE DE PERSONNES
Interdiction d'utiliser cet outil pour suivre la position d'une personne en
dehors d'une opération déclarée et limitée dans le temps. Pas de géolocalisation
en arrière-plan, pas de suivi hors mission, pas de reconstitution de trajets
individuels, pas de contrôle de l'activité ou du temps de travail d'un agent ou
d'un bénévole. Toute position cesse d'être collectée à la fermeture de
l'application et à la clôture de l'opération.

### 2. AUCUNE IDENTIFICATION BIOMÉTRIQUE NI DONNÉE SENSIBLE
Pas de reconnaissance faciale, pas de biométrie, pas de rapprochement avec des
flux vidéo ou des fichiers d'identité. Aucune donnée de santé, aucune donnée
relative à des victimes, aucune donnée nominative de personne recherchée ne doit
être saisie dans cet outil. Les secteurs de recherche sont désignés par un
identifiant géographique, jamais par le nom d'une personne.

### 3. AUCUN USAGE COMME SOURCE UNIQUE DE DÉCISION DE SÉCURITÉ
Cet outil est une aide à la coordination. Il ne remplace ni l'ordre du Commandant
des Opérations de Secours, ni la liaison radio, ni la reconnaissance de terrain.
Les données satellitaires (NASA FIRMS) présentent une latence de plusieurs heures
et une résolution de plusieurs centaines de mètres : elles constituent une
information de veille, jamais une détection opérationnelle. Aucune décision
engageant la sécurité des personnes ne doit reposer sur cette seule application.

Le responsable de traitement au sens du RGPD est l'organisation utilisatrice.
```


**Bénéfice collatéral :** ce bloc est aussi le meilleur contenu LinkedIn que vous publierez. Dans un secteur saturé de solutions de surveillance, afficher ces limites vous distingue immédiatement et ouvre des portes auprès des représentants du personnel et des DPO — deux populations capables de bloquer un projet.

---

# 9. PLAN D'ACTION — 30 PREMIERS JOURS

Ordonné par **ratio impact / effort décroissant**. Les 5 premières lignes représentent ~80 % de la valeur créée sur le mois.

| # | Action | Effort | Impact | Jour | Livrable vérifiable |
|---|---|---|---|---|---|
| **1** | **Refaire la grille tarifaire** : abonnements annuels 490 / 1 200 / 3 500 / 9 000 / 18-35 k€, formation intra à 2 400 et 4 800 €, TJM intégration à 650 €, pack tuiles à 290 €. | **0,5 j** | **Maximal** | J1-J2 | Page tarifs publiée |
| **2** | **Supprimer "détection d'incendie" de tout le discours.** Remplacer par "veille satellite — latence 3 h, résolution 375 m, en complément du 18/112". | **0,2 j** | **Maximal** | J2 | Site, README, pitch corrigés |
| **3** | **Écrire le bloc "3 interdits absolus"** dans le README (texte fourni §8.E). | **0,3 j** | **Très élevé** | J3 | README en ligne |
| **4** | **Rédiger la "fiche de registre pré-remplie" + la page Sécurité & données.** | **1 j** | **Très élevé** | J4-J5 | 2 PDF téléchargeables |
| **5** | **Lancer la recherche du co-porteur officier SP.** 15 contacts (UDSP, ENSOSP, LinkedIn, réseau perso), 4 entretiens. | **2 j** | **Maximal** | J3-J20 | 1 accord de principe |
| **6** | **Démo publique permanente** avec 3 scénarios pré-chargés (feu / RPS / DPS), testée sur un vrai Android d'entrée de gamme et un vrai iPhone. | **3 j** | **Très élevé** | J6-J10 | URL de démo + vidéo de 60 s |
| **7** | **Export PDF du zonage** (carte + sommets + trame SMS + horodatage + mention de responsabilité). | **2 j** | **Élevé** | J11-J12 | Fonction livrée |
| **8** | **Checksum + numéro de version dans la trame SMS**, et publication de la **spécification du format** en domaine public. | **1,5 j** | **Élevé** (standardisation + sécurité) | J13-J14 | `SPEC-SMS-ZONE.md` |
| **9** | **Constituer la liste de 300 prospects nominatifs** : 101 chefs de groupement SIG/SIC de SDIS, 96 présidents d'UDSP, 50 présidents d'ADPC/unités AASC, 50 DGS de communes à risque. | **2 j** | **Très élevé** | J8-J14 | Fichier CRM |
| **10** | **Ouvrir la page d'inscription au webinaire** (double créneau 12 h 30 / 20 h 30, date en S8) et lancer les invitations. | **1 j** | **Élevé** | J15-J16 | ≥ 40 inscrits à J30 |
| **11** | **Démarrer LinkedIn** : 3 posts/semaine en vidéo verticale, dès J15. | **0,3 j/sem** | **Élevé** | J15→ | 6 posts publiés à J30 |
| **12** | **Souscrire la RC Pro éditeur + cyber** et déposer la marque à l'INPI. | **0,5 j** | **Élevé** (déblocage de vente) | J17 | Attestation + dépôt |
| **13** | **Mode "exercice"** (données fictives, bandeau rouge permanent). Indispensable pour les démos et les manœuvres. | **1 j** | **Moyen-élevé** | J18-J19 | Fonction livrée |
| **14** | **Dossier de presse + vidéo de 90 s**, envoi à 10 titres spécialisés. | **1,5 j** | **Moyen-élevé** | J20-J22 | 10 envois tracés |
| **15** | **Parcours d'installation iOS guidé** + fiche A5 imprimable "mode d'emploi". | **1 j** | **Moyen** | J23-J24 | Deux livrables |
| **16** | **Instrumentation Prometheus/Grafana** (latence p99, lag de file, bufferedAmount, event loop lag) + `ARCHITECTURE.md` documentant les paliers 1/2/3. | **2 j** | **Moyen** (surtout commercial, face aux DSI) | J25-J27 | Dashboard + doc |
| **17** | **Anti-rebond d'alerte** : hystérésis 20 m + confirmation sur 2 ticks + limitation à 1 alerte / entité / zone / 5 min. | **1 j** | **Moyen-élevé** (évite le rejet produit) | J28 | Fonction livrée |
| **18** | **Identifier 3 unités pilotes** (1 CCFF, 1 ADPC, 1 commune) et proposer un exercice gratuit en S11. | **1 j** | **Très élevé** | J28-J30 | 2 accords écrits |
| | **TOTAL** | **~22 jours** | | | |

### ❌ Ce que vous ne faites PAS ce mois-ci (aussi important que la liste ci-dessus)

| À ne pas faire | Pourquoi |
|---|---|
| Le pipeline 10 Hz / Redis / sharding | Besoin réel à 50 msg/s. 3 semaines économisées. |
| Le wrapper natif / le store | Aucun client ne l'a demandé. Attendez le déclencheur. |
| Démarcher les SDIS | Cycle de 6-18 mois, aucune référence, NexSIS en embuscade. Vous y perdrez le mois. |
| Le militaire | Fermé sans habilitation. |
| Qualiopi | Attendez d'avoir vendu 3 formations pour savoir si l'axe tient. |
| Un stand en salon | 6 000 € sans référence = 0 €. Allez-y en visiteur. |
| Enrichir les fonctionnalités | **Vous avez déjà trop de produit et pas assez de clients.** Zéro nouvelle fonctionnalité non listée ci-dessus. |

---

## Les trois phrases à retenir

1. **Votre produit n'est pas une carte, c'est une trame de 35 caractères.** Tout ce qui ne sert pas à démontrer, protéger et standardiser cette trame est une distraction.
2. **Votre marché d'entrée n'est pas les SDIS, ce sont les associations, les CCFF et les communes rurales** — parce qu'ils décident en 3 semaines, paient sans marché public, n'attendent rien de l'État, et qu'ils sont peuplés des mêmes personnes qui porteront l'outil jusque dans les SDIS.
3. **Votre tarif actuel vous condamne.** 50-150 €/mois et 940 € la formation de 2 jours sont en dessous du seuil de perception budgétaire du secteur public. C'est la correction la plus rentable, et elle vous coûte une demi-journée.