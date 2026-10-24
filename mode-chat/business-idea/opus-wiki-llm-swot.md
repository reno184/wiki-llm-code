# Rôle
Tu es un stratège en business model et go-to-market, expert des modèles de
service B2B fondés sur la connaissance d'entreprise. Tu réponds en français,
de façon structurée, concrète et sans flatterie. Tu challenges mes hypothèses
quand elles sont faibles.

# Contexte : le concept "Wiki-LLM" (inspiré de l'approche de Karpathy : les
# LLM ont besoin d'un contexte de connaissances structuré pour être fiables)

## Le produit
Un "Wiki-LLM" est une base de connaissances structurée d'une organisation
(protocoles, décisions et pourquoi, checklists, snippets d'actions rares),
adossée à des LLM pour produire des agents fiables. Le point clé : ce n'est
PAS un produit technique, c'est une MÉTHODOLOGIE de formalisation du savoir
implicite, avec un format de fiche stricte de type OKF (une information vit
à un seul endroit, une fiche = un sujet, source citée, séparation
documentation/chat vs code/agent).

## Le levier de production de contenu
Les juniors et stagiaires documentent le savoir des seniors pendant leur
période d'apprentissage. Ils apprennent en documentant, le senior valide
sans écrire, l'organisation construit gratuitement son patrimoine de
connaissances. Le versionning gère le cycle qualité : branche par
stagiaire, pull request, revue, merge vers le wiki de référence.

## Stack (la partie technique est volontairement simple et interchangeable)
- Rédaction : Obsidian ou Notion (Markdown), Google Drive possible
- Révision/versioning : Gitea ou Forgejo en auto-hébergé (local chez le
  client, pilotable par API, branches + PR + merges automatisables)
- LLM : Mistral (Vibe / API), ou local avec Ollama pour les clients
  exigeant la confidentialité (santé, juridique)
- Interface : GitHub invisible pour le client — le prestataire opère la
  couche revue/merge ; le client valide en langage naturel

## Cas d'usage identifiés (valeur long terme pour les organisations)
- Chatbot support niveau 1 (base de connaissances fiable = réponses fiables)
- Onboarding juniors avec les connaissances des seniors (effet wiki + LLM
  tuteur permanent, le senior n'est sollicité que pour le jugement)
- Commerce/industrie : agents qui achètent/vendent en automatique des
  produits complexes, adossés à la base de connaissances (catalogues,
  règles fournisseurs, contrats)
- Cabinets d'avocats : automatisation des tâches répétitives adossée à la
  base de connaissances du cabinet
- Santé libérale : wiki de cabinet (protocoles, administratif), documenté
  par les stagiaires, frontière clinique stricte, hébergement local

## Mon positionnement
Je suis développeur fullstack freelance (France, TJM ~470 €). Je vends la
méthodologie, la mise en place et la revue qualité — pas de la technique.
Mon apport : la discipline de validation d'une base de connaissances (qu'est-
ce qui mérite d'entrer dans le wiki, comment le formuler pour un LLM, où
sont les frontières de responsabilité et de confidentialité).
J'envisage une association avec une amie qui a 30 ans d'expérience dans la
promotion immobilière (accès réseau, crédibilité terrain, connaissance des
flux documentaires du métier).

# Livrables attendus

## 1. SWOT du concept Wiki-LLM en tant qu'activité
- Forces, faiblesses, opportunités, menaces — spécifiques, pas génériques
- Chaque item doit être justifié en 1-2 phrases par rapport AU CONTEXTE
  ci-dessus, pas au concept abstrait de "wiki"
- SWOT global + tableau croisé Forces×Opportunités, Faiblesses×Menaces avec
  implications stratégiques concrètes

## 2. Business model — les questions qui comptent
Réponds de façon opérationnelle :
- Segmentation : quel segment choisir en PREMIER et pourquoi (santé libérale,
  immobilier/promotion, avocats, industrie/commerce, autres) — critère :
  douleur documentaire + disponibilité de stagiaires/juniors + budget +
  facilité réglementaire. Propose un classement argumenté, et le segment
  pilote optimal.
- Proposition de valeur : comment la formuler pour chaque acteur (dirigeant,
  tuteur/senior, junior/stagiaire) — elle n'est pas la même pour les trois.
- Sources de revenus : forfait setup, abonnement de maintenance/revue,
  formation, licence de la méthodologie ? Quoi vendre à qui, et dans quel
  ordre temporel (trappe possible : se vendre comme simple prestataire
  récurrent à faible valeur).
- Le rôle de l'associée immobilière : promotion immobilière comme segment
  pilote, ou comme apport de crédibilité/réseau pour d'autres segments ?
  Sois franc sur les avantages et les risques de l'association.
- Ce qui est défendable : la méthodologie est-elle copiable ? Qu'est-ce qui
  crée la barrière (revue qualité humaine, format de fiches, relation de
  confiance, historique du wiki du client, marque) ?
- Les 3 risques majeurs qui tuent l'activité, et la parade de chacun.
- Rejet du piège "produit avant preuve" : quelle est la plus petite
  validation possible (un pilote, un client, une mesure de gain) avant
  d'investiser dans l'industrialisation.

## 3. Feuille de route indicative
90 jours : pilote. 12 mois : répétition. Format : jalons, critères de
décision go/no-go entre chaque phase, chiffres de référence à mesurer
(temps senior économisé, coût du wiki, valeur d'un client répété).

# Contraintes de forme
- Tables et listes quand c'est utile, prosing narrative quand c'est mieux
- Pas de langue de bois, pas de "l'IA va tout transformer"
- Chaque recommandation doit être actionnable par un freelance seul la
  première année
- Termine par les 5 questions auxquelles je dois répondre avant de lancer
  le pilote

  ---

# Réponse Wiki-LLM — SWOT, business model et feuille de route

Avertissement de cadrage : je prends au sérieux l'hypothèse centrale (le goulot n'est pas le LLM, c'est le corpus), mais je conteste trois choses dans ton énoncé, et je le ferai explicitement : (1) « les juniors documentent gratuitement » est faux en coût complet, (2) « je vends de la méthodologie, pas de la technique » est un positionnement que ton profil ne soutient pas encore, (3) « GitHub invisible, le prestataire opère la revue » est à la fois ton fossé et ton plafond de verre.

---

## 1. SWOT du concept comme **activité** (pas comme idée)

### 1.1 Forces

| # | Force | Justification contextuelle |
|---|---|---|
| F1 | **Le levier junior transforme un coût de documentation en budget de formation** | La documentation est historiquement impayable parce qu'elle sort du budget production. Ici elle sort du budget stage/alternance, qui existe déjà et qui est déjà dépensé sans ROI mesuré. Tu ne crées pas une dépense, tu réaffectes une dépense existante. |
| F2 | **Le senior valide au lieu d'écrire : tu attaques le vrai goulot** | L'échec de 90 % des wikis d'entreprise vient du fait qu'on demande à l'expert de rédiger. Inverser la charge (il relit, il arbitre, il dit « non, pas comme ça ») consomme 5 à 10× moins de son temps et correspond à ce qu'il sait faire naturellement : corriger un junior. |
| F3 | **Le versioning importe un cycle qualité éprouvé dans un métier qui n'en a pas** | Branche / PR / revue / merge est la seule procédure de contrôle de contenu qui ait fait ses preuves à grande échelle. L'appliquer au savoir métier est un transfert technologique simple à expliquer et difficile à contester. |
| F4 | **Le format de fiche strict (OKF) est ce qui rend le corpus exploitable par un LLM** | « Une info = un endroit, une fiche = un sujet, source citée » n'est pas de l'esthétique documentaire : c'est ce qui limite la contradiction interne du corpus, premier facteur d'hallucination en RAG. C'est un argument technique que tu peux démontrer, pas seulement affirmer. |
| F5 | **Stack interchangeable = tu ne meurs pas avec un fournisseur** | Obsidian/Notion + Gitea/Forgejo + Mistral/Ollama : aucun verrou. Tu peux répondre « oui » à un client qui impose du local, du SaaS, ou du Microsoft, sans refaire ta méthode. C'est rare et c'est vendable tel quel. |
| F6 | **Tu es techniquement capable d'opérer, pas seulement de conseiller** | Un consultant pur livre un classeur et part. Tu livres un corpus qui tourne, avec l'agent branché dessus. En PME, cette capacité à aller jusqu'à la chose qui marche est la différence entre une facture payée et un pilote enterré. |

### 1.2 Faiblesses

| # | Faiblesse | Justification contextuelle |
|---|---|---|
| f1 | **Le junior ne sait pas ce qui mérite d'entrer dans le wiki** | Le tri (qu'est-ce qui est savoir réutilisable vs anecdote vs décision contextuelle) est un acte d'expertise. Un stagiaire produira 200 fiches dont 40 utiles, et personne ne saura lesquelles sans… le senior. Ton levier de production déplace le goulot, il ne le supprime pas. |
| f2 | **« Gratuit » est faux : gratification + tutorat + ton temps de revue** | Un stage > 2 mois impose une gratification (~4,35 €/h, soit ~600 €/mois), plus 2 à 4 h/semaine de tuteur, plus ta revue. Le coût réel d'un stagiaire-documentaliste sur 5 mois est de l'ordre de 6 à 9 k€ hors ta prestation. Si tu vends sur l'argument « gratuit », tu seras démasqué au premier DAF. |
| f3 | **Rotation : le savoir de documentation part avec le stagiaire** | Un stagiaire part au bout de 4-6 mois. Si la compétence « comment on écrit une fiche OKF » n'est pas internalisée chez un permanent, chaque cohorte repart de zéro et tu redeviens indispensable pour de mauvaises raisons (dépendance, pas valeur). |
| f4 | **Décalage de légitimité : un dev fullstack à 470 € vend une méthodologie de gouvernance du savoir** | Les acheteurs de méthodologie achètent à des cabinets avec des références nommées. Pendant 12 mois tu n'en auras pas. Le risque n'est pas qu'on te dise non, c'est qu'on te ramène vers ce que ton profil signale : « fais-nous le chatbot, en régie ». |
| f5 | **« Le prestataire opère la revue/merge » est un plafond de scalabilité** | Ta valeur est dans un travail humain non automatisable que tu factures au temps. À 470 €/j, ton chiffre d'affaires est borné par ton calendrier. Le fossé et le plafond sont la même chose. |
| f6 | **Aucune mesure propriétaire de la qualité aujourd'hui** | Tu promets de la fiabilité, mais tu n'as pas d'instrument pour la prouver (jeu de questions de référence, taux de bonne réponse, taux de réponse « je ne sais pas »). Sans instrument, la promesse est invérifiable, donc discutable au moment de renouveler. |
| f7 | **Solo = risque de livraison** | Un pilote de 3 mois avec un seul livreur n'a aucune redondance. Une mission dev urgente qui paie mieux tuera le pilote, et le pilote est ton seul actif commercial. |

### 1.3 Opportunités

| # | Opportunité | Justification contextuelle |
|---|---|---|
| O1 | **Les PME ont des LLM mais pas de corpus** | Tout le marché a acheté/testé un assistant en 2024-2025 et constaté qu'il répond mal sur le métier. Le diagnostic « le problème n'est pas l'outil, c'est votre documentation » arrive au moment exact où le client l'a vécu lui-même. Tu vends une explication à une déception déjà payée. |
| O2 | **Départs en retraite des sachants dans l'industrie et le bâtiment** | Les chiffreurs, méthodistes, conducteurs de travaux et techniciens SAV partent avec 30 ans de savoir non écrit. C'est une douleur datée, budgétée, avec un responsable identifié (DRH ou DG), pas un sujet « innovation ». |
| O3 | **Souveraineté / confidentialité : Mistral + Ollama en local** | Santé, juridique, défense, et de plus en plus les PME industrielles exigent que le corpus ne sorte pas. Les offres américaines y sont disqualifiées ou suspectes. C'est un argument de différenciation gratuit pour toi. |
| O4 | **L'obligation légale de transmission de savoir-faire chez les franchiseurs et réseaux** | Un franchiseur doit juridiquement transmettre un savoir-faire « identifié, substantiel et réitérable » (manuel opératoire, DIP). Ton livrable *est* leur obligation réglementaire, améliorée. Segment sous-exploité, à budget central, avec un effet réseau immédiat (1 franchiseur = N points de vente). |
| O5 | **Financement du levier junior par l'alternance** | Apprentissage = 12-24 mois (vs 5 mois de stage), aides à l'embauche, et le CFA impose un livrable écrit. Le mémoire d'alternant peut littéralement être le wiki. Tu peux co-construire l'offre avec une école et créer un canal d'acquisition. |
| O6 | **Le réseau de ton associée raccourcit le cycle de vente de 6 mois à 3 semaines** | En promotion immobilière, l'accès se fait par introduction. 30 ans de carrière = tu entres par la porte du DG, pas par le formulaire de contact. C'est l'actif le plus rare de ton dispositif. |

### 1.4 Menaces

| # | Menace | Justification contextuelle |
|---|---|---|
| M1 | **Commoditisation par les éditeurs** | Notion AI, Microsoft 365 Copilot/SharePoint, Glean, Le Chat Enterprise promettent « posez vos questions à vos documents » pour 20-30 €/utilisateur/mois. Le client n'entendra pas la différence entre « chercher dans vos docs » et « structurer un corpus fiable ». Tu dois gagner cette conversation en 90 secondes, ou tu es hors-jeu. |
| M2 | **Non-validation du senior → corpus faux → agent faux → mort de la confiance** | C'est le scénario d'échec le plus probable. Le senior est surchargé, il valide en diagonale, l'agent répond faux une fois devant le comité de direction, et le projet est mort en séance. Un seul incident suffit. |
| M3 | **Responsabilité en cas de mauvaise réponse** | Santé et juridique : si l'agent, nourri par le wiki que tu as validé, induit une erreur, la question « qui est responsable » arrive. Sans doctrine écrite de périmètre et sans RC pro adaptée, tu portes un risque sans rapport avec ta marge. |
| M4 | **Résistance culturelle des sachants** | Documenter son savoir, c'est rendre son poste substituable. Dans un cabinet d'avocats ou chez un chiffreur à 3 ans de la retraite, l'opposition sera polie et passive : on acquiesce, on ne valide jamais. |
| M5 | **Dépendance à un canal unique** | Si 100 % de ton pipe vient de ton associée, elle détient ton activité. Toute mésentente, maladie, ou changement de projet de sa part te ramène à zéro commercial. |
| M6 | **Le client recrute ton rôle en interne au bout de 18 mois** | Une fois la méthode comprise et le wiki amorcé, un responsable qualité ou un chef de projet interne peut tenir la revue. C'est la fin naturelle de ton abonnement si tu n'as pas ajouté une couche que l'interne ne sait pas faire (mesure, évolution du corpus, veille). |

### 1.5 Croisements

**Forces × Opportunités → stratégies offensives**

| Croisement | Implication stratégique concrète |
|---|---|
| F2 (senior valide) × O2 (départs retraite) | Offre nommée : **« Transfert de savoir avant départ »**. Périmètre = un sachant identifié, à horizon 12-24 mois. Déclencheur d'achat clair, budget RH/GPEC mobilisable, ROI racontable au CODIR. C'est ton meilleur angle d'entrée en industrie. |
| F3 (versioning) + F4 (OKF) × O1 (LLM déçus) | Offre de diagnostic payant : **« Audit de corpus — pourquoi votre assistant répond mal »** (3-5 k€, 3-4 jours). Tu mesures les contradictions internes, les doublons, les zones non documentées, et tu livres un plan. Ça convertit une déception en budget et ça te qualifie comme expert, pas comme dev. |
| F5 (stack interchangeable) × O3 (souveraineté) | Ne jamais vendre un outil. Vendre un **corpus portable** : « votre wiki est en Markdown versionné, il survivra à Mistral, à Microsoft et à moi ». C'est la réponse directe à M1 et un argument de PDG, pas de DSI. |
| F1 (levier junior) × O5 (alternance) | Monter un **partenariat avec 1 ou 2 écoles** (BUT/BTS, école d'immobilier, master qualité) : tu fournis la méthode et le cadre de mission, elles fournissent le flux d'alternants. Canal d'acquisition double (l'école te recommande au tuteur-entreprise). |
| F6 (tu vas jusqu'à l'exécution) × O6 (réseau associée) | Sur le pilote immobilier, livre **un agent qui marche sur un périmètre étroit en 6 semaines**, pas un wiki complet en 6 mois. L'effet démonstration dans le réseau de ton associée vaut plus que l'exhaustivité. |
| F4 (OKF) × O4 (franchiseurs) | Repackager la méthode en **« manuel opératoire augmenté »** pour franchiseurs et têtes de réseau. Même livrable, obligation légale préexistante, budget central, duplication sur N établissements. C'est ton segment de passage à l'échelle le plus probable, même s'il n'est pas ton pilote. |

**Faiblesses × Menaces → points de rupture et parades**

| Croisement | Risque réel | Parade opérationnelle |
|---|---|---|
| f1 (junior ne trie pas) × M2 (senior ne valide pas) | Corpus volumineux et faux. Scénario d'échec n°1. | **Quota de validation contractuel** : 2 créneaux de 45 min/semaine bloqués dans l'agenda du senior, signés par le DG. Règle « no validation, no merge » : aucune fiche non validée n'entre dans le périmètre de l'agent. Tu factures le créneau qu'il soit consommé ou non. |
| f6 (pas de mesure) × M1 (commoditisation) | Tu ne peux pas prouver ta supériorité sur Copilot, donc tu es comparé au prix. | **Construire un jeu d'évaluation dès le pilote** : 50 questions métier réelles + réponses de référence validées par le senior. Tu mesures avant/après et tu mesures Copilot sur le même jeu. Devient ton actif de vente n°1 et ta vraie propriété intellectuelle. |
| f5 (revue = ton temps) × M6 (internalisation) | Ton abonnement est annulé au bout de 18 mois sans que tu aies capté la valeur. | Prévoir l'internalisation **dès la vente** : l'abonnement an 1 inclut la **formation et certification d'un « éditeur de wiki » interne**. Tu passes ensuite à un contrat de supervision/audit trimestriel à plus forte marge et moins de charge. Tu choisis ta sortie au lieu de la subir. |
| f4 (légitimité) × M4 (résistance des sachants) | Tu n'as pas l'autorité pour imposer la discipline à un associé ou un chiffreur senior. | Ne jamais être le prescripteur. **Le sponsor est le DG, et c'est lui qui annonce le dispositif.** Ton rôle est opérationnel. Et commence toujours par un senior volontaire, jamais par le plus réfractaire, même si c'est lui qui détient le plus de savoir. |
| f7 (solo) × M5 (canal unique) | Un pilote, un canal, un livreur : triple point de défaillance. | **Deux pilotes en parallèle dans deux secteurs** (un via l'associée, un hors réseau), et un **actif de contenu public** (3-4 publications documentées sur la méthode et les résultats chiffrés) pour amorcer un flux entrant indépendant. |
| f2 (coût réel) × M3 (responsabilité) | Tu promets du gratuit et tu portes un risque juridique. | **Doctrine de périmètre écrite** livrée avec chaque wiki : ce que l'agent peut répondre, ce qu'il doit refuser, ce qui exige une validation humaine. Plus RC pro vérifiée pour activité de conseil. Et un budget affiché en coût complet dès la proposition commerciale. |

---

## 2. Business model

### 2.1 Segmentation : par quoi commencer

Notation 0-5 sur tes critères, plus un critère que tu as omis et qui est décisif pour un freelance seul : **l'accès** (longueur et coût du cycle de vente).

| Segment | Douleur doc. | Dispo juniors | Budget | Facilité régl. | Accès | Total /25 |
|---|---|---|---|---|---|---|
| **Promotion immobilière** | 4 | 4 | 4 | 4 | **5** | **21** |
| **Industrie / négoce technique B2B** | **5** | 4 | 4 | **5** | 2 | **20** |
| **Franchiseurs / têtes de réseau** | **5** | 3 | 4 | **5** | 2 | **19** |
| Cabinets d'avocats | 5 | 3 | **5** | 2 | 2 | 17 |
| Santé libérale | 4 | 1 | 2 | 1 | 2 | 10 |

**Lecture par segment, sans complaisance :**

- **Santé libérale : à éliminer pour l'instant.** Trois blocages cumulés. Pas de budget de structure (un cabinet libéral n'achète pas de conseil à 12 k€). Pas de stagiaires disponibles pour documenter — les internes et externes sont en soin, leur temps est médical, et leur affecter de la rédaction administrative est difficile à justifier. Et dès que tu frôles la donnée de santé, tu entres dans l'hébergement HDS et la responsabilité clinique. Le périmètre strictement administratif qui reste (protocoles d'accueil, facturation, télétransmission) a une valeur réelle mais faible, et ne paie pas ton temps. Tu l'as classé comme cas d'usage ; c'est le plus séduisant intellectuellement et le pire économiquement.

- **Avocats : excellent segment, mauvais premier segment.** La douleur et le budget sont là, mais la décision est collégiale (il faut convaincre plusieurs associés), le cycle est long, le secret professionnel ajoute une couche contractuelle, et surtout l'heure de l'associé est facturable à 250-400 € : il ne la donnera pas à de la validation de fiches tant qu'il n'a pas vu la preuve ailleurs. Garde-le pour l'année 2, avec des références en main.

- **Industrie / négoce technique B2B : ton meilleur segment de répétition.** PME de 20-200 personnes, catalogue ou process complexe, un ou deux sachants irremplaçables, aucune contrainte réglementaire, habitude d'acheter du conseil et de l'ERP, et un vivier d'alternants abondant. Le ROI est mesurable : temps de réponse sur un devis, nombre de sollicitations du chiffreur, délai de montée en compétence d'un nouveau technicien. C'est là que tu dois être en mois 6-12. Mais tu n'y as pas d'accès aujourd'hui, et le coût d'acquisition à froid est prohibitif pour un solo.

- **Franchiseurs / réseaux : l'angle mort de ton analyse, et probablement ton meilleur modèle économique.** Ils ont déjà l'obligation de transmettre un savoir-faire formalisé, ils ont déjà un budget pour le faire, et leur manuel opératoire est presque toujours un PDF mort que personne ne lit. Ton livrable est le même, mais vivant et interrogeable par chaque franchisé. Surtout : **un client = N déploiements**, ce qui casse ton plafond de temps (f5). À creuser sérieusement dès le mois 4.

- **Promotion immobilière : ton pilote.**

**Segment pilote recommandé : la promotion immobilière.** Non pas parce que la douleur y est la plus forte, mais parce que **la fonction d'un pilote n'est pas de gagner de l'argent, c'est de produire une preuve, vite et à coût d'acquisition nul.** Tu as un accès direct, crédible et chaleureux via ton associée ; en industrie tu passerais 4 mois à obtenir trois rendez-vous. Un pilote qui démarre en semaine 3 vaut dix fois un meilleur pilote qui démarre en mois 6.

Trois conditions pour que ce pilote serve à quelque chose au-delà de lui-même :
1. **Choisir un périmètre réitérable**, pas un périmètre spécifique à l'opération en cours. Les meilleurs candidats : le **montage d'opération** (checklist de faisabilité, pièces, interlocuteurs collectivité, points de blocage récurrents et leur raison) ou le **SAV / GPA** (réserves types, qui est responsable de quoi, réponses aux acquéreurs). Le SAV/GPA est plus facile à mesurer et moins politique.
2. **Mesurer dès le jour 1** : temps senior consommé par semaine sur des questions répétitives, avant et après.
3. **Écrire le livrable comme un cas transposable**, structuré pour être rejoué dans un autre secteur : c'est ce qui transforme un client en offre.

### 2.2 Proposition de valeur, par acteur

La même phrase ne marche pas pour les trois. Et il y en a un quatrième que tu as oublié : le DAF, qui signe.

| Acteur | Ce qu'il craint | Formulation | Preuve à lui apporter |
|---|---|---|---|
| **Dirigeant** | Que le savoir de l'entreprise soit dans la tête de 3 personnes dont 2 partent | « Votre entreprise ne possède pas son propre savoir-faire : trois personnes le louent. On en fait un actif qui reste, financé par votre budget alternance, et qui devient la base de vos assistants. » | Le chiffre : combien d'heures de vos sachants partent chaque semaine en questions déjà posées. |
| **Tuteur / senior** | Devoir écrire, et devenir remplaçable | « Vous n'écrirez pas une ligne. Vous corrigez un junior 45 minutes deux fois par semaine, comme vous le faites déjà — sauf que cette fois ça s'accumule, et qu'on arrête de vous interrompre dix fois par jour pour la même question. » | La liste des questions répétitives qu'il ne recevra plus. Et jamais le mot « remplacer ». |
| **Junior / alternant** | Être affecté à une tâche ingrate, sans apprentissage | « Tu vas apprendre le métier en interrogeant le meilleur du cabinet, et tu repars avec une production signée, revue, et un mémoire déjà écrit. Documenter un métier est la façon la plus rapide de l'apprendre. » | Le cadre : fiches signées, revue formelle, compétence transférable (structuration de connaissance pour IA). |
| **DAF / contrôle de gestion** | Une ligne budgétaire « IA » sans retour | « Le budget est connu à l'avance, le livrable est un actif portable que vous possédez, et on mesure le résultat sur un jeu de questions défini à l'avance. S'il n'est pas atteint, on ne passe pas en phase 2. » | Le go/no-go écrit, chiffré, daté. |

### 2.3 Sources de revenus : quoi vendre, à qui, dans quel ordre

**La trappe que tu as toi-même identifiée est la bonne, et elle est pire que tu ne le penses :** si tu commences par vendre de la revue de PR mensuelle, tu deviens un sous-traitant documentaire à 1 000 €/mois, remplaçable par une alternante en communication, et tu auras plafonné à 5 clients. L'ordre temporel est donc une décision stratégique, pas une commodité.

| Offre | Prix indicatif | Marge / scalabilité | Quand l'introduire |
|---|---|---|---|
| **1. Audit de corpus** (3-4 j) | 3 500 – 5 000 € | Haute, réplicable | Dès maintenant. C'est ton produit d'appel et ton outil de qualification. |
| **2. Sprint de formalisation** (pilote, 6-8 semaines, périmètre fermé) | 10 000 – 15 000 € forfait | Moyenne la 1re fois, haute dès la 3e | Immédiatement après l'audit. Jamais en régie. |
| **3. Abonnement de gouvernance** (revue, QA de l'agent, rapport mensuel) | 900 – 2 000 €/mois | Faible (ton temps) mais c'est la récurrence et le verrou | Seulement après le sprint, et contractuellement limité à 12 mois reconductibles. |
| **4. Formation + certification d'un éditeur interne** (2 j intra) | 4 000 – 6 000 € | Haute | Mois 6+. C'est ce qui te fait sortir du rôle d'exécutant. |
| **5. Supervision / audit trimestriel** (après internalisation) | 1 200 – 2 000 € / trimestre | Très haute | Mois 15+. Remplace l'abonnement. |
| **6. Licence de méthode + kit** (templates, grille OKF, jeu d'éval, doctrine de périmètre) à d'autres freelances/cabinets | 3 000 – 8 000 €/an | Très haute, mais nécessite une marque | An 2 minimum. Ne pas y penser avant 3 cas documentés. |

**Séquence recommandée :** audit → sprint → abonnement court → formation → supervision. La formation est le pivot : c'est elle qui transforme ta faiblesse f5 (ta valeur = ton temps) en produit, et qui anticipe M6 (internalisation) au lieu de la subir.

**Deux règles de tarification :**
- **Ancrer sur le coût du temps senior, jamais sur ton TJM.** Un sachant à 90 €/h chargé, 5 h/semaine récupérées = ~20 k€/an. Un forfait à 12 k€ se justifie en une ligne. Un devis en « 25 jours × 470 € » invite à la négociation sur les jours.
- **Tout en forfait, avec périmètre fermé et quota de validation senior contractualisé.** Si le quota n'est pas tenu par le client, le périmètre se réduit — pas ton prix.

**Réalité comptable de l'année 1, puisque tu veux du concret :** 2 audits (8 k€) + 2 sprints (24 k€) + 3 abonnements démarrant en moyenne au mois 8 (≈ 15 k€) + 1 formation (5 k€) ≈ **50-55 k€**, pour un temps consommé d'environ 60-70 jours de livraison et autant de jours commerciaux non facturés. C'est en dessous de ce que te rapporte une année de dev à 470 €. **Conclusion : l'année 1 se finance par ton activité de dev, à 50 % de ton temps maximum.** Si tu ne tiens pas cette contrainte, le premier trou de trésorerie te renverra en régie et le pilote mourra. Planifie-le maintenant.

### 2.4 Le rôle de l'associée : apport de réseau, pas co-fondation

**Ma réponse franche : promotion immobilière comme terrain de pilote, et elle comme ouvreuse de portes — mais pas comme associée à parts égales au départ.**

Ce qu'elle apporte est réel et difficilement remplaçable : accès au décideur, crédibilité pour poser les bonnes questions à un directeur de programme, compréhension des flux documentaires (montage, appels d'offres, VEFA, livraison, GPA), et capacité à détecter ce qui mérite d'être documenté — c'est-à-dire précisément la faiblesse f1, celle qui fait échouer le dispositif.

Les risques, en revanche :

| Risque | Pourquoi c'est sérieux | Montage de protection |
|---|---|---|
| **Déséquilibre de contribution** | Son apport est concentré au début (ouvrir des portes) ; le tien s'étale sur toute la durée (livrer, maintenir). Une association 50/50 dès le jour 1 te fait payer pendant 5 ans un apport de 6 mois. | Commence par un **contrat d'apporteur d'affaires** : 10-15 % du CA sur les affaires qu'elle amène, pendant 24 mois. Révisez vers une société commune quand il y a un vrai fonds de commerce à partager. |
| **Enfermement sectoriel** | Son réseau est immobilier. Si elle est associée, elle orientera naturellement la stratégie vers son secteur, qui n'est pas ton meilleur segment de répétition. | Écrire noir sur blanc que l'immobilier est le **terrain de preuve**, pas le marché cible. Fixer un objectif chiffré de clients hors immobilier d'ici le mois 12. |
| **Confusion des rôles en clientèle** | Deux interlocuteurs non hiérarchisés devant un DG = flottement sur qui engage quoi. | Partage explicite : elle ouvre et co-anime le cadrage, tu es seul responsable du périmètre, du devis et du livrable. |
| **Sortie non prévue** | Les ruptures d'association entre amis sont la première cause de mort de micro-structures. | Clause de sortie, propriété de la méthode et des templates à ton nom dès maintenant, et pacte écrit avant le premier client commun — pas après. |

Résumé : **utilise-la à 100 % comme canal et comme caution métier, structure la relation comme un apport d'affaires rémunéré, et garde la propriété de la méthode.** Si au mois 9 elle a généré 3 clients et co-animé les cadrages, l'association devient une évidence et se négociera sur des faits.

### 2.5 Ce qui est défendable

**La méthodologie seule n'est pas défendable.** Ton format de fiche OKF tient en 4 pages ; un concurrent le copie en une journée, et un LLM peut le régénérer. Si tu construis ton discours sur « ma méthode », tu perds. Classement honnête de tes barrières, de la plus faible à la plus forte :

| Barrière | Solidité | Commentaire |
|---|---|---|
| Format de fiche / méthode | **Faible** | Copiable en un week-end. À publier plutôt qu'à protéger : la diffuser te positionne comme auteur, ce qui vaut plus que de la cacher. |
| Marque | **Faible an 1** | Inexistante aujourd'hui. Se construit par les cas publiés, pas par un logo. |
| Relation de confiance | **Moyenne** | Réelle mais personnelle et non transmissible. Elle protège un client, pas une activité. |
| Historique du wiki client (coût de transfert) | **Forte, mais c'est la barrière du client, pas la tienne** | Le corpus appartient au client. Il le protège d'un changement d'outil, il ne te protège pas d'un remplacement par un interne. Ne confonds pas les deux. |
| **Corpus de templates sectoriels éprouvés** | **Forte** | 40 fiches-types « montage d'opération », « SAV/GPA », « chiffrage », « onboarding technicien » issues de cas réels : ça, ça ne se régénère pas, ça s'accumule. C'est ton vrai capital. |
| **Le jeu d'évaluation et la discipline de mesure** | **La plus forte** | Savoir construire le jeu de 50 questions qui prouve qu'un agent est fiable sur un métier donné, mesurer le taux de bonne réponse et le taux d'abstention, et le refaire à chaque release : personne en PME ne sait le faire, et c'est la seule chose qui transforme « je crois que ça marche » en « voici le chiffre ». Fais-en ton produit. |
| **La doctrine de périmètre et de responsabilité** | **Forte en secteur régulé** | Le document qui dit ce que l'agent a le droit de répondre, ce qu'il doit refuser, et où passe la frontière. C'est ce que les avocats et la santé achèteront, et ce que les éditeurs ne livreront jamais. |

**Reformule ton positionnement en conséquence :** tu ne vends pas une méthodologie de documentation, tu vends **la fiabilité mesurée d'un agent sur un périmètre métier, et la frontière de responsabilité qui va avec.** Le wiki et la méthode sont les moyens.

### 2.6 Les trois risques qui tuent l'activité

**Risque 1 — Le senior ne valide pas.** C'est le plus probable et le plus létal : sans validation, le corpus est du bruit, l'agent se trompe, et le projet meurt en comité. Il ne se manifeste pas par un refus mais par un glissement poli de créneaux.
*Parade :* quota contractuel de validation signé par le DG (2 × 45 min/semaine), règle « no validation, no merge », périmètre de l'agent strictement limité aux fiches validées, et **KPI hebdomadaire public : nombre de fiches validées cette semaine**. Si deux semaines consécutives sont à zéro, tu arrêtes le pilote et tu le dis. Arrêter un pilote qui dérive te coûte moins cher que de le livrer mauvais.

**Risque 2 — Commoditisation par les éditeurs.** Le client compare ton forfait de 12 k€ à 25 €/utilisateur/mois de Copilot et conclut que tu es cher.
*Parade :* ne jamais te positionner contre l'outil, toujours **en amont** de l'outil. Argumentaire en une phrase : « Copilot cherche dans vos documents. Si vos documents se contredisent, il vous répond avec assurance quelque chose de faux. Je produis le corpus qui rend Copilot fiable — et je le prouve avec un jeu de questions avant/après. » Puis tu fais la démonstration : fais tourner leur assistant actuel sur 20 questions métier réelles devant eux. Le taux d'erreur fait la vente à ta place. Et techniquement : intègre-toi à leur outil plutôt que d'en imposer un.

**Risque 3 — Dépendance à un canal unique et absence de preuve transposable.** Un pilote, un réseau, un livreur : n'importe quelle défaillance te ramène à zéro.
*Parade :* deux pilotes en parallèle dans deux secteurs dès le trimestre 1, un contrat d'apport d'affaires qui n'engage pas l'avenir, et surtout la production d'un **actif public** : un cas chiffré et anonymisé publié, plus 3-4 articles sur la méthode. L'objectif mois 9 est d'avoir au moins une demande entrante qui ne vienne pas de ton réseau. Si tu ne l'as pas, tu n'as pas une activité, tu as un client.

*(Risque 4, à surveiller sans en faire une priorité : la responsabilité juridique si tu entres en santé ou en juridique. Vérifie ta RC pro et écris ta doctrine de périmètre avant, pas après.)*

### 2.7 La plus petite validation possible

Tu as raison de rejeter le « produit avant preuve ». Voici le plus petit dispositif qui tranche vraiment.

**Un client. Un périmètre. Six semaines. Un chiffre.**

- **Périmètre :** un seul processus douloureux et répétitif (ex. SAV/GPA en promotion immobilière), choisi parce qu'il génère des questions récurrentes mesurables.
- **Volume cible :** 30 à 40 fiches validées. Pas 200. Un corpus petit et juste bat un corpus large et douteux — et c'est toute la démonstration.
- **Instrument de mesure, défini AVANT de commencer :** 50 questions métier réelles, collectées dans les mails/Teams/appels des 3 derniers mois, avec les réponses de référence validées par le senior. Tu mesures trois choses : taux de réponse correcte, **taux d'abstention correcte** (l'agent dit « je ne sais pas » quand il ne sait pas — c'est le vrai indicateur de fiabilité), taux d'erreur avec assurance.
- **Mesure d'impact :** nombre d'interruptions hebdomadaires du senior sur ce périmètre, avant et après.
- **Budget :** 8 à 12 k€ facturés, ~20 jours de ton temps.
- **Seuil de succès à annoncer d'avance :** ≥ 80 % de réponses correctes sur le jeu, ≤ 5 % d'erreurs avec assurance, et ≥ 30 % d'interruptions senior en moins.

Si ces trois seuils sont atteints, tu as une offre. S'ils ne le sont pas, tu as appris où ça casse pour 20 jours — et tu n'as rien industrialisé.

---

## 3. Feuille de route

### Phase 0 — Semaines 1 à 3 : armement (avant tout client)

| Jalon | Livrable |
|---|---|
| J+7 | Grille OKF v1 : gabarit de fiche, règles de découpage, règle de source, règles d'exclusion (ce qui n'entre PAS dans le wiki). 4 pages maximum. |
| J+10 | Doctrine de périmètre v1 : ce que l'agent répond, ce qu'il refuse, ce qui exige un humain. 2 pages. |
| J+14 | Chaîne technique prête et rejouable : Obsidian + Forgejo + RAG sur Mistral API et sur Ollama local. Monte-la une fois pour toutes, chronomètre-la : objectif < 1 journée par client. |
| J+18 | Protocole de mesure : modèle de jeu d'évaluation 50 questions + feuille de calcul des 3 taux. **C'est ton actif le plus important, soigne-le.** |
| J+21 | Contrat d'apport d'affaires signé avec l'associée + kit commercial (1 page de diagnostic, 1 proposition type d'audit, 1 proposition type de sprint). |

**Go/no-go P0 → P1 :** la chaîne technique est rejouable en moins d'une journée **et** 3 rendez-vous de qualification sont calés. Sinon, tu n'es pas prêt à vendre.

### Phase 1 — Jours 21 à 90 : le pilote

| Semaine | Jalon | Critère de passage |
|---|---|---|
| S4-5 | **Audit de corpus** chez le client pilote (facturé 3-5 k€) : cartographie des sources, contradictions, zones non documentées, questions récurrentes. | Le client paie l'audit. S'il refuse de payer 4 k€ pour un diagnostic, il ne paiera pas 12 k€ pour un sprint. **Test de qualification décisif.** |
| S6 | **Cadrage du sprint** : périmètre fermé, jeu de 50 questions construit, quota de validation senior signé par le DG, seuils de succès écrits. | Quota signé. Sans signature du DG, tu ne démarres pas. |
| S7-12 | **Sprint de formalisation** : junior/alternant produit, senior valide en créneaux, tu opères branches/PR/merge, l'agent est branché sur le corpus validé uniquement. KPI hebdo affiché. | ≥ 5 fiches validées par semaine en rythme de croisière. Deux semaines à zéro = arrêt et restitution. |
| S13 | **Restitution chiffrée** devant le DG : 3 taux mesurés, interruptions senior avant/après, coût complet réel du dispositif, plan de phase 2. | Les 3 seuils (80 % / 5 % / −30 %) sont atteints. |

**Go/no-go P1 → P2 — trois conditions cumulatives, pas une sur trois :**
1. Les seuils de mesure sont atteints.
2. Le client signe un abonnement de gouvernance (même à 900 €/mois) — c'est le seul vrai vote.
3. Le senior dit spontanément que ça lui a fait gagner du temps. Si tu dois le lui faire dire, c'est non.

Si une seule condition manque : tu rejoues un deuxième pilote dans un autre secteur avec les corrections. Tu n'industrialises rien.

### Phase 2 — Mois 4 à 12 : répétition

| Échéance | Jalon | Chiffre de référence à atteindre |
|---|---|---|
| M4 | 2e pilote lancé **hors réseau de l'associée** (cible : négoce/industrie technique, angle « transfert de savoir avant départ en retraite »). | Cycle de vente mesuré. Objectif : < 8 semaines de la prise de contact à l'audit signé. |
| M5 | Cas n°1 publié (anonymisé, chiffré) + 2 articles de méthode. | 1re demande entrante d'ici M9. |
| M6 | **Offre de formation** packagée : « éditeur de wiki certifié » (2 j intra). Vendue au client pilote. | 1 formation vendue à 4-6 k€. |
| M7 | Partenariat avec 1 école (alternance) pour sécuriser le flux de documentalistes. | 2 alternants placés chez des clients. |
| M8 | Exploration du segment franchiseurs / têtes de réseau : 5 rendez-vous de découverte. | 1 audit signé. |
| M9 | Industrialisation du **coût de production d'une fiche validée**. | Objectif : < 25 € par fiche en coût complet (gratification + tutorat + ta revue). Au-dessus de 60 €, le modèle ne tient pas. |
| M10-12 | 3 abonnements de gouvernance actifs, 1 passage en supervision trimestrielle. | CA activité ≈ 50-60 k€. Taux de renouvellement : 100 % des abonnements arrivés à échéance. |

**Chiffres de référence à mesurer en permanence (ton tableau de bord, 6 lignes) :**

| Indicateur | Cible an 1 | Pourquoi |
|---|---|---|
| Temps senior économisé / semaine / client | ≥ 3 h | C'est l'unité de valeur. Tout ton pricing en découle. |
| Coût complet d'une fiche validée | < 25 € | Détermine si le levier junior est réel ou s'il est une fiction comptable. |
| Taux de bonne réponse sur jeu d'éval | ≥ 85 % | Ta preuve face aux éditeurs. |
| Taux d'erreur avec assurance | ≤ 3 % | Le seul taux qui tue un projet. Prioritaire sur le précédent. |
| Valeur client cumulée sur 24 mois | ≥ 35 k€ | Audit + sprint + abonnement + formation. En dessous de 25 k€, ton coût d'acquisition solo n'est pas couvert. |
| Part du CA issue du réseau de l'associée | ≤ 60 % à M12 | Mesure directe du risque M5. |

**Go/no-go M12 → an 2 :** trois clients ayant payé un sprint **et** renouvelé un abonnement, dans au moins deux secteurs différents, avec une mesure d'impact reproductible. Si oui, l'an 2 porte sur la licence de méthode, le segment franchise et le recrutement d'un premier relais. Si non, tu as une très bonne prestation de niche à exercer seul — ce qui est un résultat acceptable, à condition de l'assumer plutôt que de forcer une industrialisation.

---

## Les 5 questions auxquelles tu dois répondre avant de lancer le pilote

1. **Qui est le senior nommé, et son DG a-t-il signé par écrit 2 × 45 min de validation par semaine pendant 8 semaines ?** Tant que cette signature n'existe pas, tu n'as pas de pilote, tu as un espoir. C'est la variable la plus létale de tout le dispositif.

2. **Quelles sont les 50 questions, et qui détient les réponses de référence ?** Si tu ne peux pas les collecter en 3 jours dans les mails et les conversations du client, c'est que la douleur n'est pas là où tu crois — et que tu vas documenter ce qui est facile plutôt que ce qui coûte cher.

3. **Le client acceptera-t-il de payer 4 000 € pour l'audit seul, avant tout engagement sur la suite ?** C'est le seul test de solvabilité et de sérieux qui compte. Un « on verra après le POC gratuit » est un non déguisé, et tu dois le traiter comme tel.

4. **Quelle est ta réponse en 90 secondes à « on a déjà Copilot / Notion AI » — et peux-tu la démontrer, pas l'affirmer ?** Si tu n'as pas préparé la démonstration du taux d'erreur de leur outil actuel sur leurs vraies questions, tu perdras cette conversation à chaque rendez-vous, quelle que soit la qualité de ta méthode.

5. **Combien de jours de dev factures-tu en parallèle, et as-tu bloqué les 20 jours du pilote dans ton calendrier comme non négociables ?** L'année 1 se finance par ton activité existante. Si le plan de charge n'est pas arbitré maintenant, la première mission urgente à 470 €/j arbitrera à ta place, et le pilote mourra sans que tu aies rien appris.