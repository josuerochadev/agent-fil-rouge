# Exercice 4 — Jouer avec les paramètres du LLM
Matériel : accès à l'API OpenAI ou Claude (via playground, curl ou SDK Python)
____________________________________________________________
## Le prompt de référence
Utilisez ce prompt pour TOUTES les variations :
**"Écris un email professionnel pour annoncer un retard de livraison de 2 semaines à un client important. Le produit est un logiciel de gestion RH. Le client est le DRH d'un grand groupe."**
## Variation 1 — Température basse
Paramètres : temperature = 0.1, max_tokens = 300
Exécutez 3 fois de suite avec les mêmes paramètres.
* Les 3 réponses sont-elles identiques ou différentes ?
* Le ton est-il formel / neutre / créatif ?

V1 (temp basse) : les 3 emails sont quasi identiques en structure — même plan en 4 paragraphes, mêmes tournures ("nous sommes conscients de l'importance", "à votre entière disposition"). La température basse confirme la stabilité, mais aussi la monotonie.

## Variation 2 — Température haute
Paramètres : temperature = 0.9, max_tokens = 300
Exécutez 3 fois de suite.
* Les 3 réponses varient-elles beaucoup entre elles ?
* Y a-t-il des formulations surprenantes ou inappropriées ?

V2 (temp haute) : les 3 emails varient davantage dans l'accroche et la conclusion — le run 1 ajoute une invitation explicite à un "point d'avancement", le run 3 mentionne une "nouvelle date" concrète. Le fond reste professionnel, GPT-4o ne déraille pas même à 0.9 sur ce type de prompt factuel.

## Variation 3 — Tokens limités
Testez avec max_tokens = 50 puis max_tokens = 500.
* À 50 tokens, l'email est-il coupé en plein milieu d'une phrase ?
* À 500 tokens, le modèle ajoute-t-il du contenu superflu ?

V3 (50 tokens) : coupure nette en plein milieu de la phrase "en raison de circonstances indépendantes" — exactement le comportement attendu. À noter que GPT-4o choisit de commencer directement l'email sans le préambule "Bien sûr ! Voici...", signe que la limite de tokens force une économie immédiate.

V3 (500 tokens) : l'email est plus complet, propose un "suivi régulier jusqu'à la livraison finale" — contenu utile, pas vraiment superflu sur ce sujet.

## Variation 4 — Frequency penalty
Testez avec frequency_penalty = 0.0 puis frequency_penalty = 1.5.
* Avec 0 : voyez-vous des répétitions ?
* Avec 1.5 : le texte est-il plus varié ou bizarre ?
## Tableau de synthèse

| Variation | Température | Max tokens | Freq penalty | Ton | Qualité /5 | Exploitable ? |
| ------------- | ----------- | ---------- | ------------ | -------------------------------------------------- | ---------- | ---------------------- |
| T basse | 0.1 | 300 | 0 | Formel, neutre, très stable | 4/5 | Oui |
| T haute | 0.9 | 300 | 0 | Formel mais plus varié, légèrement plus chaleureux | 3.5/5 | Oui |
| Tokens courts | 0.3 | 50 | 0 | Formel mais incomplet | 1/5 | Non — coupé mid-phrase |
| Tokens longs | 0.3 | 500 | 0 | Formel, plus détaillé, propose un suivi régulier | 4.5/5 | Oui |
| Sans penalty | 0.5 | 300 | 0 |  |  |  |
| Penalty forte | 0.5 | 300 | 1.5 |  |  |  |

 
## Livrable
* Tableau de synthèse rempli
* Réglage optimal recommandé pour cet email : temperature=___, max_tokens=___, frequency_penalty=___

> temperature=0.2, max_tokens=350, frequency_penalty=0.0

* Règle générale en 1 phrase : "Pour un usage professionnel factuel, je règle...

> "Pour un usage professionnel factuel, je règle la température entre 0.1 et 0.3 pour garantir cohérence et ton formel, max_tokens entre 300 et 400 pour laisser la place à un email complet sans contenu superflu, et frequency_penalty à 0 car le risque de répétition est faible sur des textes courts et structurés."
