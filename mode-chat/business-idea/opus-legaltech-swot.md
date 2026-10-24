# Question (prompt)

# Rôle
Tu es un investisseur/rédacteur de "killing floor" : ta seule mission est de trouver les failles. Tu n'es PAS là pour encourager. Interdiction de tempérer chaque critique par un compliment. Si une partie du modèle est morte, dis "morte" et explique pourquoi. Pas de langage corporate.

# Contexte (tout est vrai, ne le remets pas en question, attaque le modèle)
2 associés. Fondateur 1 : développeur fullstack senior freelance (Vue/Nuxt/Node + Python), France. Fondateur 2 : formatrice IA, très bonne présentation à l'écran, ~50 ans, 30 ans d'expérience en gestion de projet immobilier — ATTENTION : elle n'est PAS avocate, pas juriste.
Contraintes du binôme :
- sans employés, sans rendez-vous de vente, sans démos en visio en continu
- tunnel 100% self-service : vidéo explicative sur landing page + cold emailing → achat en ligne
- one-shot au départ, pas d'abonnement

# Le produit
Outil d'anonymisation/pseudonymisation de PDF pour avocats, juristes, DRH, notaires, M&A.
Stack : Python (PyMuPDF) + IA locale (Ollama). Déploiement : Docker sur un VPS OVH cloud acheté pour le client ; Ollama tourne en local dans le Docker et l'application y accède par URL. Confidentialité obligatoire (secret professionnel des avocats) — aucune API cloud sur les documents.
Règle d'architecture : l'IA locale ne fait QUE des tâches simples ciblées (pas de traitement de documents longs de bout en bout).

# Orchestration technique prévue
- Découpage des documents en chunks, orchestré par des workflows d'automatisation (type workflows.js / Vercel) : les tâches IA locale tournent en fond, le client est prévenu de fin de tâche par email, push ou websocket.
- Hypothèse matériel client : machines 8 Go RAM, au mieux 16 Go sur Mac. Modèle : Qwen3.5:9b → l'IA locale est LENTE.
- Parallélisation des tâches : pendant que l'IA locale chunk + anonymise, elle cherche en parallèle les clauses "red flag" M&A dans les mêmes documents. Objectif : quand le banquier d'affaires envoie le contrat anonymisé à Claude (IA cloud, sur document anonymisé donc conforme), il a déjà le résultat de l'IA locale et peut comparer les deux analyses.

# Architecture actée (suite tests réels)
Le 100% automatique local n'est PAS atteignable sur documents longs (modèle 9B en local : catastrophe au-delà de 4 pages) → human-in-the-loop systématique : l'utilisateur sélectionne les zones et valide l'extraction. Tesseract et Docling écartés (résultats décevants).

# Features futures prévues
Splitting de PDF par lots avec nommage intelligent, extraction → Excel avec liens vers le bloc source, détection de signatures, recherche transversale de clauses, chronologie automatique des faits.

# Le modèle de vente envisagé (ATTAQUE-LE)
- Formation en ligne à 299 € (one-shot), le client installe/utilise lui-même
- Vidéos ultra-détaillées, chat intégré au produit, Slack dédié, "la totale"
- 3 mois de support offerts
- Objectif affiché : vendre en masse (10 clients/jour cités comme ambition), pas d'onboarding individuel
- Plus tard : ajouter d'autres features au même outil pour augmenter la valeur

# Peurs exprimées
- Ne pas pouvoir onboarder 10 clients/jour ; prise en main à distance ingérable
- Charge de support d'un produit déployé en VPS Docker (OVH) : gestion de la flotte de VPS, disponibilité, mises à jour

# Mission — SWOT non conciliant
Produis un SWOT impitoyable, puis des verdicts.

## 1. SWOT complet (Strengths / Weaknesses / Opportunities / Threats)
Attaque AU MINIMUM avec ces questions :
- VPS OVH géré par le fondateur : 10 clients/jour = une flotte de VPS à provisionner, monitorer, mettre à jour, sécuriser (patches, CVE Ollama/Docker), facturer OVH… qui gère ça et combien de temps ça prend ? "Aucune installation chez le client" = déplacer le problème, pas le résoudre ?
- Positionnement "secret professionnel respecté" : le VPS OVH est-il défendable comme "local" ? Il est dans le cloud OVH, pas chez le client — un avocat prudent peut-il considérer que la chaîne de confidentialité est respectée ? Qui a accès au VPS (le fondateur = un tiers de plus qui touche les documents) ?
- Qwen3.5:9b sur 8 Go RAM = lent : quelle expérience client réelle ? Un cabinet accepte-t-il d'attendre des dizaines de minutes par document avec une notification par email ?
- Le binôme : la formatrice n'est ni avocate ni juriste — crédibilité du contenu de formation juridique ? Risque de "conseil juridique non sollicité" ? Sa force est la présentation vidéo, mais le canal est le cold email : où apparaît-elle dans le tunnel ?
- "Formation 299 €" : qui paie une formation pour apprendre à utiliser un outil ? Ne faut-il pas vendre l'OUTIL avec la formation incluse ? 299 € est-il compatible avec le coût d'acquisition d'un client avocat par cold email ?
- Support 3 mois offerts + chat + Slack : un abonnement déguisé ? Que coûte réellement 1 h de support par client si on en vend 200 ? Le VPS ajoute du support d'infra au support logiciel.
- Contradiction architecture : human-in-the-loop obligatoire partout vs promesse de self-service sans accompagnement. Le client a-t-il le temps de faire ce travail de review ?
- Double analyse (IA locale vs Claude sur document anonymisé) : belle idée, mais qui croit l'IA locale lente 9B quand elle contredit Claude ? Quelle est la valeur réelle du recoupement pour le client ?
- Cold emailing vers des avocats : délivrabilité, aucune preuve sociale, tunnel sans humain — taux de conversion réaliste ?
- Concurrence : Adobe Acrobat Pro fait de la redaction correcte pour ~25 €/mois sans rien installer. Pourquoi le client choisirait ce produit ?
- Responsabilité : anonymisation ratée = fuite de données sous secret professionnel. Qui assume le risque juridique si le masquage rate une PII ? Le binôme a-t-il une assurance RC pro couvrant ce cas ?

## 2. Les 5 causes de mort les plus probables
Classées par probabilité, avec un pourcentage estimé de risque d'échec du business à 12 mois.

## 3. Le modèle est-il SAUVABLE ?
Si tu ne devais garder qu'UNE variable modifiable (prix, cible, mode de déploiement, canal, périmètre produit, rôle de l'associée) pour maximiser les chances, laquelle et pourquoi.

## 4. 3 variantes de business model alternatives plus robustes
Pour un binôme qui déteste le support — dont au moins une qui supprime totalement la gestion d'infrastructure par le fondateur tout en restant conforme au secret professionnel, et au moins une qui exploite la force réelle de l'associée (présence, formation, 30 ans de gestion de projet). Sois créatif : offre de service premium, partenariats, niche ultra-ciblée, etc.

# Format
Titres nets, listes denses, aucune phrase de remplissage. Chaque critique se termine par une implication opérationnelle concrète ("donc : ...").

---

# REPONSE KILLING FLOOR — Analyse sans filtre

---

## 0. Diagnostic d'entrée (avant le SWOT)

Vous vendez **trois produits différents** à **une cible qui n'achète aucun des trois de cette manière** :
1. Un logiciel (l'outil d'anonymisation)
2. Une infogérance (le VPS OVH)
3. Une formation (299 €)

Et vous facturez le troisième en espérant que le client paie implicitement pour les deux premiers. C'est le point de rupture central. Tout le reste en découle.

---

## 1. SWOT IMPITOYABLE

### STRENGTHS (courtes, et plus fragiles qu'elles n'en ont l'air)

- **Compétence technique réelle et rare** : fullstack senior + Python + orchestration IA locale. Vous pouvez *construire*. C'est 20 % du problème.
  *→ donc : votre avantage est en production, pas en distribution. Vous avez optimisé la partie que vous maîtrisez déjà.*
- **Human-in-the-loop acté suite à tests réels** : vous avez tué votre propre fantasme du 100 % auto. Rare et sain.
  *→ donc : vous avez une vision honnête du produit — mais vous n'avez pas encore réécrit la promesse commerciale en conséquence. Voir Weaknesses.*
- **Angle confidentialité** : c'est le seul vrai différenciateur face aux SaaS US.
  *→ donc : si cet angle tombe (et il tombe, voir ci-dessous), il ne reste rien contre Adobe.*
- **Features futures (extraction → Excel avec lien bloc source, chronologie des faits, recherche transversale de clauses)** : ça, c'est de la valeur métier. Plus que l'anonymisation.
  *→ donc : votre roadmap vaut mieux que votre V1. Problème de séquencement, voir §3.*
- **Zéro dette, zéro salarié, zéro investisseur** : vous pouvez pivoter en 48 h.
  *→ donc : la seule raison de ne pas pivoter serait l'ego.*

### WEAKNESSES (le gros du morceau)

**W1 — Le VPS OVH est une bombe à retardement opérationnelle**
- 10 clients/jour = 200/mois = **200 VPS à provisionner, monitorer, patcher, sauvegarder, facturer**. À 15 min/VPS/mois de maintenance moyenne (optimiste, hors incident), c'est 50 h/mois de pur ops. À 500 clients, c'est un temps plein. Vous n'avez pas d'employés, par décision.
- CVE Docker/Ollama : une faille critique = 200 VPS à patcher **en urgence, le même jour**, sinon vous êtes l'hébergeur négligent de documents sous secret professionnel.
- Ollama a déjà eu des CVE d'exécution de code à distance et d'exposition de port non authentifié. Un `0.0.0.0:11434` mal fermé sur un VPS public et vous avez un incident de sécurité sur des pièces de procédure.
- Qui paie OVH ? Si c'est le client via son compte : vous lui demandez de créer un compte cloud, prendre une CB, suivre un tuto de création de VPS. **Taux d'abandon massif dans un tunnel self-service**. Si c'est vous : vous avez une trésorerie récurrente, de la facturation, de la TVA, des impayés — et un abonnement de fait alors que vous vendez du one-shot à 299 €.
- **Le one-shot à 299 € ne couvre même pas 18 mois de VPS** (un VPS capable de faire tourner un 9B avec un minimum de confort, c'est 20–60 €/mois, et sans GPU c'est lent à pleurer ; avec GPU, c'est 150 €+/mois).
*→ donc : votre marge unitaire est **négative** dès le mois 10, et tend vers moins l'infini à mesure que vous vendez. Vous avez construit une machine à perdre de l'argent proportionnellement au succès.*

**W2 — "Aucune installation chez le client" = vous êtes devenu hébergeur, pas éditeur**
- Vous ne supprimez pas le problème, vous le transférez du client vers vous, dans le sens le plus coûteux possible : du côté où il n'y a que 2 personnes dont une non-technique.
- Vous devenez de fait **sous-traitant au sens RGPD (art. 28)** : DPA obligatoire, registre, obligation de notification de violation sous 72 h, obligation de sécurité démontrable, audit possible par le cabinet.
*→ donc : il vous faut un DPA type, un registre, une politique de sécurité écrite et une procédure d'incident **avant la première vente**. C'est 3 à 5 k€ d'avocat IT ou 3 semaines de votre temps. Vous ne l'avez pas budgété.*

**W3 — "Secret professionnel respecté" sur un VPS que VOUS administrez : positionnement indéfendable**
- Un VPS OVH **n'est pas "local"**. C'est du cloud. Marketer ça comme "IA locale, rien ne sort" face à un avocat déontologiquement formé = vous donnez à votre prospect le plus intelligent une raison de ne pas vous faire confiance **et** de le dire à ses confrères.
- Vous avez un accès root. Vous êtes donc **un tiers supplémentaire qui peut techniquement lire les pièces d'un dossier en cours**. L'avocat doit en informer son client ou au minimum le couvrir contractuellement. Le bâtonnier peut avoir un avis.
- Le réflexe du prospect avocat sérieux : "Qui a les clés ? Où sont les logs ? Qu'est-ce qui est chiffré au repos ? Que se passe-t-il si vous êtes réquisitionné ou si vous arrêtez l'activité ?" Sans réponse écrite, c'est non.
- Le pire : **votre argument de vente #1 est aussi votre plus gros risque juridique.** Le jour où vous écrivez noir sur blanc "conforme au secret professionnel", vous prenez un engagement que vous ne pouvez pas tenir techniquement seul.
*→ donc : soit vous passez en on-premise/desktop réel (zéro accès de votre part), soit vous arrêtez d'utiliser le mot "local" et vous assumez "hébergement souverain français" — ce qui est un argument beaucoup plus faible et très concurrencé.*

**W4 — La performance : le produit est inutilisable au rythme du métier**
- Qwen 9B sur CPU (VPS OVH sans GPU) : on parle de **quelques tokens/seconde**. Un document de 40 pages découpé en chunks = **des dizaines de minutes à plusieurs heures**.
- Votre propre constat : "catastrophe au-delà de 4 pages". Or la réalité d'un cabinet : des PDF de 80, 200, 600 pages (data rooms M&A, bordereaux de pièces, dossiers prud'homaux).
- Le pattern "lance le job → reçois un email → reviens valider" est un pattern de **batch industriel**, pas d'outil de bureau. Un avocat qui prépare une audience veut un résultat en 90 secondes, pas une notification à 23 h.
- Et comme il y a human-in-the-loop, **le délai machine s'additionne au délai humain** au lieu de le remplacer.
*→ donc : mesurez et publiez le temps réel sur un dossier de 50 pages. Si c'est > 10 min, votre marché n'est pas "les avocats", c'est "les traitements de nuit" — et ça change toute la promesse.*

**W5 — Contradiction frontale : human-in-the-loop obligatoire VS self-service sans accompagnement**
- Human-in-the-loop = l'utilisateur sélectionne les zones et valide. Autrement dit : **le client fait le travail**. Sur 200 pages, c'est 2 h de clics pour un professionnel facturé 150–350 €/h. **Le coût d'usage de votre outil dépasse son prix d'achat dès le deuxième dossier.**
- L'avocat ne déléguera pas ça à sa secrétaire s'il y a un enjeu de responsabilité. Et s'il le délègue, c'est à un stagiaire qui le fait déjà au Stabilo pour 0 €.
- Un outil qui exige de l'apprentissage + de la discipline + de la validation manuelle est **l'anti-produit self-service**. Le self-service ne fonctionne que sur les produits à "time to value" < 10 minutes.
*→ donc : soit vous automatisez pour de vrai (NER spécialisé fine-tuné, pas un 9B généraliste prompté), soit vous facturez le travail humain — c'est-à-dire vous vendez un service, pas un logiciel.*

**W6 — "Formation à 299 €" : la pire structure de prix possible**
- **Personne n'achète une formation pour apprendre à utiliser un outil qu'il ne possède pas encore.** Vous vendez le mode d'emploi avant le produit. C'est inversé.
- 299 € pour un cabinet : ce n'est pas un montant "trop cher", c'est un montant **sans statut**. Trop cher pour un achat impulsif à la CB sans réflexion (ce seuil est ~50–100 €), trop peu cher pour déclencher une évaluation sérieuse et un sentiment de garantie sur un sujet aussi sensible que le secret pro. Vous êtes dans la vallée de la mort tarifaire.
- Un avocat qui voit "299 € pour anonymiser des PDF" pense : "mon stagiaire le fait, Acrobat le fait, pourquoi ?"
- Et vous appelez ça formation pour des raisons probablement fiscales/OPCO — sauf que **sans certification Qualiopi, aucun financement OPCO**. Donc vous avez les inconvénients du mot "formation" (froideur, obligation de contenu, concurrence de milliers de formations IA à 97 €) sans l'avantage (le financement).
*→ donc : vendez une **licence logicielle** avec formation incluse, à 1 500–3 000 € pour un cabinet, ou 79–149 €/mois. Le mot "formation" doit disparaître du pricing.*

**W7 — Économie unitaire du cold email : le 299 € ne tient pas**
Chiffres réalistes du cold email B2B vers avocats en 2026 :
- Délivrabilité : les domaines de cabinets filtrent agressivement. Taux de délivrance réel 70–85 % au mieux, avec warm-up et domaines secondaires.
- Taux d'ouverture : 25–40 %. Taux de réponse : **1–3 %**. Taux de conversion réponse → achat **sans aucun appel humain, sur un produit technique sensible, sans preuve sociale** : 2–5 % des répondants.
- Donc : **~0,05 % d'emails → vente**. Pour 10 ventes/jour il faut ~20 000 emails/jour. C'est industriellement impossible pour 2 personnes sans tomber dans le spam pur, le blacklistage de domaine, et une plainte CNIL.
- Et le marché est fini : il y a ~75 000 avocats en France. Vous "brûlez" votre TAM en quelques mois de cold email mal ciblé.
- Coût d'acquisition réaliste par client en cold email sur cette cible : **150 à 600 €**. Avec un prix de 299 € TTC (≈ 249 € HT), vous êtes **à l'équilibre ou en perte avant même d'avoir payé le VPS et le support**.
*→ donc : à 299 €, le cold email est mathématiquement mort. Soit le prix monte d'un facteur 5–10, soit le canal change.*

**W8 — "10 clients/jour" est un chiffre fantasmé, et il est structurellement incompatible avec tout le reste**
- 10 clients/jour = 2 600/an = plus de clients que la plupart des legaltech françaises en ont après 5 ans et une levée de fonds.
- Et chacun apporte : 1 VPS + un accès Slack + du chat + 3 mois de support.
- **À 2 600 clients, même 20 min de support par client par an = 866 h/an.** Soit un mi-temps permanent, gratuit, sur un revenu one-shot déjà encaissé et dépensé.
*→ donc : ce chiffre n'est pas un objectif, c'est un scénario de faillite. Si vous l'atteignez, vous mourez plus vite que si vous échouez.*

**W9 — Support "3 mois offerts + chat + Slack + la totale" = abonnement déguisé, non facturé, non bornable**
- Vous offrez une obligation récurrente contre un paiement unique. C'est la définition d'un passif.
- Un Slack communautaire d'avocats ayant des problèmes d'infra sur un outil touchant le secret pro : **c'est un amplificateur d'insatisfaction publique**, pas un canal de support. Le premier client dont le VPS tombe le dit devant les 200 autres.
- À J+91, vous coupez le support ? Sur un produit qui nécessite human-in-the-loop et tourne sur un VPS que vous administrez ? **Vous ne couperez jamais.** Vous le savez.
*→ donc : le support doit être soit payant et borné (SLA écrit), soit remplacé par du produit (le support est un bug de conception, pas une feature).*

**W10 — L'associée : hors du tunnel, et exposée**
- Sa force : **présence à l'écran, pédagogie, 30 ans de gestion de projet immobilier**. Votre tunnel : cold email → landing page → CB. **Il n'y a littéralement aucun endroit où sa force s'exprime**, sauf une vidéo sur la landing. C'est du gâchis de 50 % de votre capital humain.
- Son profil ne crédibilise pas une cible avocat : ni avocate, ni juriste, ni issue du droit. Devant un bâtonnier ou un associé de cabinet d'affaires, "formatrice IA + immobilier" ne crée pas l'autorité. Ça la crée devant **des agences immobilières, des promoteurs, des syndics, des notaires, des gestionnaires de patrimoine** — pas devant un avocat M&A.
- Risque réel : une formation qui explique *quelles données doivent être anonymisées pour être conforme* est à un cheveu du **conseil juridique**, monopole des avocats (loi du 31 déc. 1971, art. 54/60). Dire "voici quoi masquer pour être RGPD-conforme" = exercice illégal du droit, réprimé pénalement.
*→ donc : soit elle change de cible (immobilier/notarial, où elle est légitime), soit tout contenu normatif est purgé et remplacé par du "comment utiliser le logiciel", ce qui réduit la formation à un tuto — qui ne vaut pas 299 €.*

**W11 — La double analyse (IA locale 9B vs Claude) ne crée pas de valeur, elle crée de la charge mentale**
- Le banquier d'affaires reçoit deux analyses contradictoires. Que fait-il ? **Il croit Claude.** Toujours. Parce qu'il sait que le 9B local est un modèle 20× plus petit.
- Donc l'analyse locale sert à… être ignorée, après avoir consommé du temps de calcul et retardé le livrable.
- Pire : la seule chose que l'analyse locale pourrait apporter (un "deuxième avis" indépendant) n'a de valeur que si elle est **au moins aussi fiable**. Elle ne l'est pas. Un faux négatif du 9B n'informe personne ; un faux positif ajoute du bruit à trier.
- Et commercialement : "notre IA détecte les red flags de votre contrat" est une **promesse de conseil juridique**. C'est un aimant à responsabilité.
*→ donc : tuez la feature "red flags" en V1, ou repositionnez-la en pure checklist déterministe (regex/patterns de clauses : changement de contrôle, exclusivité, non-concurrence, earn-out), sans IA, sans interprétation, et appelée "repérage", jamais "analyse".*

**W12 — Responsabilité sur l'anonymisation ratée : trou béant**
- Une PII manquée = violation du secret professionnel + violation RGPD. Le client est responsable devant son client ; il se retournera contre vous.
- Vos CGV limiteront votre responsabilité — mais une limitation de responsabilité **ne tient pas** face à une faute lourde, et sur un produit dont c'est la fonction unique, un juge regardera attentivement.
- Avez-vous une **RC Pro éditeur de logiciel + cyber** ? Qui couvre les dommages immatériels non consécutifs ? Prime réaliste : 1 500–4 000 €/an minimum sur ce secteur. **C'est 5 à 13 ventes à 299 € juste pour l'assurance.**
- Et le vrai piège technique : PyMuPDF. Si le masquage est fait par un rectangle dessiné et non par une vraie suppression du contenu texte (`apply_redactions`), **le texte reste extractible sous le rectangle**. C'est l'erreur historique qui a grillé des cabinets et des administrations. Un seul cas public et votre marque est morte sur un marché de 75 000 personnes qui se parlent toutes.
*→ donc : audit de non-régression obligatoire sur chaque export (re-extraction du PDF de sortie + vérification que zéro entité détectée n'y subsiste), et RC Pro souscrite avant la première vente. Non négociable.*

**W13 — Vous avez écarté Tesseract et Docling, donc vous n'avez pas d'OCR**
- Une part énorme des pièces en cabinet est **scannée** (jugements, pièces adverses, contrats signés, actes notariés). Sans OCR, ces documents sont des images : vous ne détectez rien, l'utilisateur doit tout cadrer à la main.
- "Résultats décevants" n'est pas une raison de supprimer la fonction, c'est une raison de changer de brique (AWS Textract est exclu pour confidentialité ; restent PaddleOCR, Surya, docTR, ou un VLM local — tous plus lourds que Qwen 9B).
*→ donc : soit vous assumez "PDF natifs uniquement" en gros sur la landing (et vous perdez ~40 % des cas d'usage), soit vous rouvrez le chantier OCR. Le silence sur ce point est un mensonge par omission qui produira des remboursements.*

### OPPORTUNITIES (réelles, mais pas là où vous regardez)

- **L'angle "souveraineté / pas d'IA américaine"** est un argument commercial puissant en France en 2026, particulièrement auprès des ordres, des barreaux, des chambres de notaires et des DPO. Marché politique autant que technique.
- **La douleur réelle n'est pas l'anonymisation**, c'est : "j'ai 1 400 pages de data room et 3 jours". Les features que vous avez reléguées au futur (**splitting par lots + nommage intelligent, extraction → Excel avec lien vers le bloc source, chronologie automatique des faits**) valent 10× l'anonymisation en perception de valeur. La chronologie des faits, en particulier, est le Graal pour un contentieux.
- **Vendre à une structure, pas à un individu** : un barreau, un réseau de cabinets, un groupement de notaires, une école (EFB/CRFPA), un éditeur legaltech existant. Une vente = 50 à 500 utilisateurs. Supprime le besoin de 10 clients/jour.
- **Le marché notarial** : 7 000 offices, très standardisés, forte pression de dématérialisation, et **moins allergiques au prestataire externe** que les avocats. Et c'est le terrain naturel de l'associée (immobilier).
- **Les DRH** : anonymisation de CV (discrimination à l'embauche), dossiers disciplinaires, enquêtes internes. Cible moins exigeante déontologiquement, budget formation réel, et **cycle de vente plus court**.
- **Expertise rare monétisable directement** : "déploiement d'IA locale conforme en cabinet" est une prestation de conseil à 1 000–1 500 €/jour que le Fondateur 1 peut vendre demain, sans produit.

### THREATS

- **Adobe Acrobat Pro, ~25 €/mois** : redaction réelle, Sanitize Document, suppression des métadonnées, recherche-et-masquage de motifs, déjà installé, déjà validé par la DSI, déjà budgété. **Votre réponse à "pourquoi pas Acrobat ?" doit tenir en une phrase. Si elle n'existe pas, le produit est mort.** (La seule réponse valable : "Acrobat ne trouve pas les noms *pour vous*, il masque ce que vous lui désignez". Mais votre human-in-the-loop annule justement cette différence.)
- **Le commodity risk** : dans 12–18 mois, l'anonymisation est une case à cocher dans tous les outils legaltech (Doctrine, Jarvis, Ordalie, Lexis+ AI, Predictice) et dans les suites bureautiques. C'est une **feature, pas un produit**.
- **Les modèles locaux progressent plus vite que vous** : un 4B de 2027 battra votre 9B d'aujourd'hui. Bonne nouvelle technique, mauvaise nouvelle stratégique : votre différenciateur (faire tourner du local) devient trivial pour tout le monde.
- **Un incident de sécurité unique vous tue.** Marché de 75 000 personnes, bouche-à-oreille ultra-dense, barreaux qui diffusent des alertes. Pas de seconde chance.
- **CNIL + cold emailing B2B** : le B2B tolère l'opt-out, mais l'email d'un avocat exerçant en nom propre est une donnée personnelle. Une campagne massive mal faite = plainte + mise en demeure.
- **Le risque de burnout du binôme** : 1 dev, 1 non-tech, un produit à 3 composants (logiciel + infra + pédagogie), un canal froid, et aucune récurrence. Le vrai risque n'est pas l'échec commercial, c'est **l'abandon au mois 7** quand le freelance reprend une mission à 600 €/jour parce que le produit a généré 4 000 € en 6 mois.
- **Le coût d'opportunité est votre plus gros ennemi** : le Fondateur 1 a un taux journalier. Chaque mois passé sur ce produit coûte 8 000–12 000 € de CA non facturé. À 299 €, il faut **vendre 35 formations par mois juste pour égaler le freelance qui ne fait rien de spécial.**

---

## 2. LES 5 CAUSES DE MORT LES PLUS PROBABLES

Risque global d'échec à 12 mois, en l'état actuel du modèle : **~88 %**. Décomposition :

| # | Cause de mort | Probabilité (12 mois) | Mécanisme |
|---|---|---|---|
| **1** | **Acquisition inexistante** : le cold email ne convertit pas à 299 €, aucune preuve sociale, pas d'humain dans le tunnel, cible la plus sceptique du B2B français. | **55 %** | Vous ne faites pas 10 ventes/jour, vous faites 2 ventes/mois. Au mois 6, le CA cumulé est < 5 000 €. Le Fondateur 1 reprend une mission freelance. Mort par essoufflement, pas par échec bruyant. C'est le scénario majoritaire. |
| **2** | **Asphyxie ops/support** : si ça marche, la flotte de VPS + le support non borné consomme 100 % du temps du Fondateur 1. | **20 %** | Paradoxe mortel : ce scénario n'arrive que si le #1 n'arrive pas. Marge unitaire négative, zéro temps de dev, churn invisible (produit one-shot = le client disparaît sans le dire). Mort par succès. |
| **3** | **Objection confidentialité non levée** : le VPS tiers administré par vous est rejeté par les prospects les plus sérieux ; l'argument "local" est démonté en 2 questions. | **8 %** | Chaque prospect qualifié pose la question d'accès root et vous perdez exactement les clients qui avaient le budget. Les seuls qui achètent sont ceux qui n'ont pas compris — donc ceux qui vous feront un litige plus tard. |
| **4** | **Incident de masquage / sécurité** : une PII qui fuite, un PDF mal rédacté, un port Ollama exposé. | **3 %** | Faible probabilité à faible volume, **impact terminal**. Réputation détruite sur un marché clos + exposition en responsabilité sans assurance. C'est le seul risque qui peut vous coûter plus que le projet. |
| **5** | **Rupture du binôme** : asymétrie d'effort (le dev porte produit + infra + support ; l'associée n'a pas de place dans un tunnel sans humain), absence de revenus, démotivation. | **2 %** | Le "sans rendez-vous de vente" prive l'associée de son seul terrain de jeu. Elle devient spectatrice d'un produit technique. Séparation au mois 8. |

> Les % sont des probabilités de **cause première** de la mort, conditionnelles au fait qu'elle survienne (88 %).

---

## 3. LE MODÈLE EST-IL SAUVABLE ? — UNE SEULE VARIABLE

**Oui, mais pas tel quel. La variable unique à changer : LA CIBLE.**

Pas le prix. Pas le déploiement. **La cible.**

### Pourquoi la cible et rien d'autre

Changer le prix sans changer la cible : vous vendez 1 500 € à un avocat qui ne répond déjà pas à 299 €. Zéro effet.

Changer le déploiement (desktop) sans changer la cible : vous résolvez l'ops, mais vous gardez un CAC supérieur au prix. Vous mourez quand même, juste plus proprement.

Changer le canal sans changer la cible : les avocats sont inatteignables en froid sans preuve sociale ni humain. Vous payez juste plus cher pour le même non.

**La cible, elle, débloque tout en cascade :**

| Variable | Avec "avocats" | Avec la bonne cible |
|---|---|---|
| Objection confidentialité | Bloquante (secret pro, déontologie, bâtonnier) | Gérable (RGPD standard, DPA suffit) |
| Crédibilité de l'associée | Nulle (pas juriste) | **Maximale** (30 ans d'immobilier / gestion de projet) |
| Prix acceptable | 299 € perçu cher | 2 000–5 000 € perçu normal |
| Nombre de clients nécessaires | 2 600/an | **30 à 60/an** |
| Besoin de support de masse | Écrasant | Maîtrisable |
| Preuve sociale | Inaccessible | Un logo = toute la filière |

### La cible à prendre : **les notaires et/ou les acteurs de la transaction immobilière professionnelle** (études notariales, promoteurs, foncières, asset managers, syndics, et les cabinets d'avocats *immobiliers* comme extension ultérieure)

Raisons, sans sentiment :
1. **L'associée y est légitime à 100 %.** 30 ans de gestion de projet immobilier. Elle connaît les acteurs, le vocabulaire, les process, probablement des gens. Elle passe de "formatrice IA générique" à "experte métier avec un outil". **C'est le seul scénario où elle vaut 50 % de la boîte.**
2. **Les documents sont standardisés** : actes, baux, promesses, compromis, états datés, diagnostics, PV d'AG, titres de propriété. Standardisé = automatisable = votre 9B redevient suffisant car les tâches sont ciblées et répétitives. **Vous sortez du "document long arbitraire" qui tue votre archi.**
3. **La douleur est réelle et chiffrable** : une data room d'acquisition d'immeuble, c'est 300 baux à dépouiller pour en sortir un tableau Excel (locataire, surface, loyer, indexation, échéance, clause de sortie). **C'est exactement votre feature "extraction → Excel avec lien vers le bloc source".** Aujourd'hui c'est fait à la main par un junior en 3 semaines à 400 €/jour. Vous vendez 15 000 € d'économie. **Là, 3 000 € de licence n'est plus une discussion.**
4. **Pas de secret professionnel d'avocat** → le VPS devient défendable (RGPD + DPA + hébergement France suffisent).
5. **Marché atteignable** : 7 000 offices notariaux, quelques centaines de foncières/asset managers, syndics structurés. Vous pouvez le couvrir nommément. Pas besoin de 20 000 emails/jour.
6. **Les avocats restent accessibles plus tard**, par la porte de service : une fois que vous avez 20 références immobilières, vous entrez chez les avocats en droit immobilier avec de la preuve sociale et un cas d'usage prouvé. **L'ordre des facteurs est votre seule erreur stratégique majeure.**

### Ce que ça impose, concrètement

- *donc :* l'anonymisation n'est plus le produit principal — c'est **l'extraction structurée de données contractuelles immobilières**, avec anonymisation en option/conformité.
- *donc :* l'associée reprend la main sur la distribution (réseaux, chambres, syndicats professionnels, FNAIM, CSN, salons RENT/SIMI), et sa caméra sert enfin à quelque chose.
- *donc :* le prix passe à 2 000–4 000 € par étude/structure, avec maintenance annuelle 20 %. 40 clients = 100 k€. Objectif atteignable par un binôme.
- *donc :* le cold email devient du **cold email nominatif ultra-ciblé** (500 contacts bien choisis), pas de la masse.

---

## 4. TROIS VARIANTES DE BUSINESS MODEL PLUS ROBUSTES

### Variante A — **"Appliance desktop"** : zéro infrastructure gérée par le fondateur, confidentialité réellement défendable

**Le principe :** vous ne touchez plus jamais un document client. L'application s'installe sur le poste ou sur le serveur **du client**, derrière son pare-feu. Vous vendez un binaire + une licence.

- **Forme technique** : un installeur unique (Tauri ou Electron + sidecar Python compilé + Ollama embarqué, ou bundle Docker Desktop avec script one-click). Modèle téléchargé à la première ouverture. Licence par clé signée, vérification offline avec grâce de 30 jours.
- **Si le poste client est trop faible (8 Go)** : option "serveur de cabinet" — un mini-PC fourni clés en main (Mac mini M4 16 Go ≈ 700 €, ou mini-PC Ryzen avec iGPU) préconfiguré, expédié, branché sur le réseau du cabinet. **Vous vendez une boîte, pas un abonnement cloud.** Marge matérielle + licence. Et l'argument devient imparable : *"le document ne quitte jamais vos locaux, nous n'avons aucun accès, il n'y a pas de compte à créer"*.
- **Prix** : 1 490 € licence perpétuelle poste unique / 3 900 € licence cabinet (5 postes) / +900 € pour l'appliance matérielle. Maintenance et mises à jour : 18 %/an optionnel.
- **Support** : forums + base de connaissance + **1 h d'installation assistée facturée 250 €** (et non offerte). Le support n'est plus un passif, c'est une ligne de CA.
- **Ce que ça tue** : la flotte de VPS, la facturation OVH, les CVE à patcher en urgence, le statut de sous-traitant RGPD, l'objection "qui a accès ?", la marge négative.
- **Ce que ça coûte** : packaging multi-OS, gestion des licences, mises à jour par les clients (acceptable : vous poussez un updater).
- *donc :* c'est la variante à construire **par défaut**. Le VPS OVH n'a qu'un seul avantage — éviter de packager — et il vous coûte votre modèle économique entier. **Deux semaines de packaging valent mieux que 200 VPS.**

---

### Variante B — **"Cabinet de production documentaire assisté"** : vous vendez le résultat, pas l'outil

**Le principe :** vous arrêtez de vendre un logiciel que le client doit apprendre, installer et opérer. **Vous faites le travail**, avec votre outil comme avantage de coût interne. Le client envoie des documents, reçoit un livrable.

- **Offre** : "Data room prête en 72 h" — anonymisation + découpage + nommage normalisé + tableau de synthèse Excel avec lien vers le bloc source + chronologie des faits. Livrable : un ZIP + un XLSX.
- **Prix** : 3 à 8 € par page traitée, ou forfait 2 500 € par dossier jusqu'à 500 pages, 6 000 € jusqu'à 2 000 pages. Urgence 48 h : +50 %.
- **Pourquoi c'est plus robuste** :
  - **Le human-in-the-loop devient un atout** au lieu d'une contradiction : c'est *vous* l'humain, et c'est justement ce que le client paie.
  - **La lenteur du 9B devient invisible** : le client a un délai de 72 h annoncé, vos jobs tournent la nuit sur *une seule* machine que vous maîtrisez (un seul GPU, pas 200 VPS).
  - **Zéro onboarding, zéro formation, zéro installation, zéro support logiciel.**
  - Vous validez le marché et le besoin **avant** d'écrire la V2. Chaque mission vous dit quelle feature construire.
  - Le CAC de 300–600 € est amorti sur un panier de 2 500 €+ et sur des dossiers répétés.
- **Confidentialité** : traitement sur machine dédiée en France, NDA + DPA, collaborateurs = les 2 fondateurs uniquement. **Un cabinet accepte déjà des prestataires externes** (traducteurs, huissiers, imprimeurs juridiques, e-discovery) : c'est une catégorie d'achat qui existe, contrairement à "logiciel d'anonymisation à installer".
- **Le piège à éviter** : c'est une agence, ça ne scale pas linéairement. **Mais c'est exactement le point** — vous n'avez pas besoin de scaler, vous avez besoin de 15 dossiers/mois à 3 000 €. Et vous automatisez progressivement votre propre back-office jusqu'à ce que le produit soit assez bon pour être vendu en self-service (Variante A) **à des clients qui vous connaissent déjà**.
- *donc :* c'est la variante qui génère du cash le plus vite, avec zéro risque d'infra et zéro support. À lancer **en premier**, même si l'objectif final est A.

---

### Variante C — **"Programme de conformité IA pour la filière immobilière & notariale"** : l'associée est le produit

**Le principe :** le logiciel n'est pas vendu, il est **inclus**. Ce qui est vendu, c'est un programme accompagné, porté par l'associée, à destination des structures.

- **Cible** : études notariales, groupements (CSN, chambres départementales), réseaux d'agences, promoteurs, syndics, foncières, et plus tard cabinets d'avocats en droit immobilier.
- **Offre "Cabinet IA-conforme" — 6 900 € par structure** :
  1. Audit documentaire (2 demi-journées, en visio ou sur place) : cartographie des flux de documents sensibles. **Terrain natif de l'associée : 30 ans de gestion de projet, elle sait lire une organisation.**
  2. Déploiement de l'appliance (Variante A) : installée, configurée, testée. **Terrain du Fondateur 1.**
  3. Programme de formation 4 × 2 h en visio live, en groupe, pour toute l'équipe — porté par l'associée. **Sa présence à l'écran est enfin monétisée au bon endroit.**
  4. Kit documentaire : politique interne d'usage de l'IA, registre, clauses types pour les lettres de mission/mandats, procédure d'incident. **Co-signé par un avocat partenaire** — ce qui règle d'un coup le risque d'exercice illégal du droit ET apporte la caution que l'associée n'a pas.
  5. Suivi trimestriel 1 h pendant 12 mois.
- **Renouvellement annuel : 1 200 €** (mises à jour logicielles + veille réglementaire + 1 session de formation). **Voilà votre récurrence, et elle est justifiée par de la valeur, pas par du support subi.**
- **Distribution** : pas de cold email de masse. **Partenariats prescripteurs** — un avocat partenaire (qui apporte la caution juridique et ses clients), les chambres professionnelles, les experts-comptables spécialisés immobilier, les salons de la filière (SIMI, RENT, Congrès des Notaires). 20 % de commission d'apport.
- **Économie** : 30 structures/an × 6 900 € = **207 k€**, + 36 k€ de récurrent en année 2. Pour 2 personnes, sans employé, sans flotte de VPS.
- **Charge de support** : bornée par construction (suivi trimestriel planifié), et l'appliance supprime l'infra.
- *donc :* c'est la seule variante où le binôme est équilibré, où le CAC est couvert, où l'associée est indispensable, et où le risque juridique est externalisé à un partenaire qualifié.

---

## Synthèse exécutable

| | Garder | Tuer immédiatement |
|---|---|---|
| **Cible** | Notarial / immobilier professionnel / DRH | Avocats en cible primaire (les garder en cible 2, après références) |
| **Déploiement** | Appliance locale (desktop ou mini-PC chez le client) | **La flotte de VPS OVH** — c'est la décision la plus destructrice du plan |
| **Prix** | 2 000–7 000 € par structure + récurrent de maintenance | **Les 299 €** et le mot "formation" dans le pricing |
| **Produit** | Extraction structurée → Excel avec lien source, splitting + nommage, chronologie | **La détection de "red flags" par IA** (promesse de conseil juridique, valeur nulle face à Claude) |
| **Canal** | Partenariats prescripteurs + 500 contacts nominatifs + présence filière | **Le cold email de masse** vers 20 000 avocats |
| **Associée** | Audit d'organisation + formation live en groupe + relation filière | **Son absence du tunnel** |
| **Support** | Payant, borné, planifié, contractualisé (SLA écrit) | **Le Slack ouvert + chat + "la totale" offerts** |

**Les trois actions des 30 prochains jours, dans l'ordre :**
1. **Vendre 3 dossiers en Variante B** (service, à la page) à des contacts du réseau de l'associée. Prix plein. Avant toute ligne de code supplémentaire. Si vous n'en vendez pas 1, le besoin n'existe pas au prix où vous croyez.
2. **Écrire le test de non-régression du masquage** : re-extraire chaque PDF de sortie et échouer le build si une entité détectée y subsiste. Sans ça, vous ne devez vendre à personne.
3. **Supprimer le VPS du plan.** Packager une appliance. C'est 2 semaines de travail contre une vie entière d'ops.

Et la question à laquelle vous devez répondre en une phrase avant tout le reste : **"Pourquoi pas Acrobat Pro à 25 €/mois ?"**
Si la réponse n'est pas évidente pour un prospect en 5 secondes de lecture, rien d'autre dans ce document n'a d'importance.