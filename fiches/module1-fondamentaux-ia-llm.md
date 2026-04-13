# Module 1 — Fondamentaux IA & LLM

> Formation : *Maitriser les LLM et l'IA Generative* — AJC Formation

---

## 1. Hierarchie de l'IA

L'evolution de l'IA suit une **progression technologique logique**, chaque etape surmontant les limites de la precedente :

| Niveau | Description |
|---|---|
| **IA classique** | Systemes experts codes manuellement, regles explicites |
| **Machine Learning** | Les modeles apprennent des patterns statistiques a partir de donnees |
| **Deep Learning** | Reseaux neuronaux profonds (images, voix, texte) |
| **IA Generative** | Modeles qui creent du contenu (texte, images, code, audio) |

### Types de Machine Learning

- **Supervise** : apprentissage a partir d'exemples etiquetes (classification, regression)
- **Non supervise** : decouverte de structures cachees (clustering, reduction de dimensionnalite)
- **Par renforcement** : apprentissage par essai-erreur avec recompenses

---

## 2. Tokens et fenetre de contexte

### Tokens

- Unite de base des LLM : sous-mots, pas des mots complets
- Le francais consomme **~25% de tokens en plus** que l'anglais (impact direct sur les couts)
- 1 token ≈ 3/4 de mot en francais

### Fenetre de contexte (Context Window)

Taille maximale de texte (entree + sortie) que le modele peut traiter en une seule requete.

| Modele | Contexte |
|---|---|
| GPT-4o | 128K tokens |
| Claude 3.7 | 200K tokens |
| Gemini 1.5 | 1M tokens |

**Strategies en cas de depassement** : chunking, resume, RAG.

---

## 3. Hallucinations et biais

### Hallucinations

Information **inventee, incorrecte ou non verifiable** produite par le modele avec une apparente confiance.

**Exemples** :
- **Citations fictives** : references de jurisprudence inventees (cas reel USA 2023 — avocat sanctionne)
- **Bibliographie inventee** : etudes et auteurs inexistants
- **Code plausible mais incorrect** : fonctions avec des signatures fictives
- **Erreur factuelle** : "Thomas Edison a invente l'electricite" (association statistique erronee)

**Cause** : le modele predit ce qui est linguistiquement probable, sans verifier la realite factuelle.

### Biais

Reponse **systematiquement influencee** par des tendances desequilibrees dans les donnees d'entrainement.

**Exemples** :
- **Recrutement** : sous-representation des femmes dans les metiers techniques reproduite par le modele
- **Generation d'images** : "PDG d'entreprise" → homme blanc, 50 ans, costume sombre
- **Biais culturel** : "grands philosophes" → uniquement occidentaux (omet Confucius, Avicenne, Nagarjuna, Sankara)

**Cause** : le modele reproduit et amplifie les inegalites de son corpus.

### Hallucination vs Biais

| | Hallucination | Biais |
|---|---|---|
| Definition | Information inventee | Reponse systematiquement desequilibree |
| Exemple | Citation inexistante | Associer un metier a un genre |
| Cause | Prediction linguistique sans verification | Reproduction des desequilibres du corpus |

### Pourquoi ces phenomenes existent

- **Prediction statistique** : les LLM predisent le prochain token le plus probable, ils ne raisonnent pas
- **Absence de base de verite** : aucun mecanisme de verification factuelle integre
- **Reproduction des patterns** : structures, associations et desequilibres du corpus d'entrainement

### Bonnes pratiques

**Reduire les hallucinations** :
- Demander les sources et les verifier
- Utiliser le RAG pour ancrer les reponses dans des documents verifies
- Demander une auto-verification ("Es-tu certain ?")

**Reduire les biais** :
- Tester les modeles sur des cas representatifs de populations diverses
- Diversifier les donnees d'entrainement
- Auditer les resultats avec des outils dedies

### A retenir

> **Les LLM ne sont pas des moteurs de verite.** Ils sont des moteurs de probabilite linguistique — capables de produire des reponses utiles, fluides et convaincantes, mais parfois incorrectes ou biaisees.

- **Tres utiles** : synthese, redaction, brainstorming, assistance au code
- **A verifier** : faits precis, chiffres, references, dates, noms propres
- **A auditer** : decisions automatisees (recrutement, justice, medecine)

---

## 4. Ecosysteme LLM

### Principaux acteurs

| Acteur | Positionnement |
|---|---|
| **OpenAI** | Leader mondial, modeles GPT, ecosysteme API tres developpe |
| **Anthropic** | Focus securite IA, modeles Claude reconnus pour leur fiabilite |
| **Google** | Gemini, forte integration ecosysteme Google |
| **Mistral** | Open source, souverainete europeenne |
| **Hugging Face** | Plateforme open source, hub de modeles |

### Criteres de selection

- **Performance vs Cout** : le plus puissant n'est pas toujours le plus rentable
- **Confidentialite & Souverainete** : proprietaire (heberge a l'etranger) vs open source (deployable en local)
- **Ecosysteme & Integrations** : APIs, plugins, compatibilite avec l'existant

### Comparaison des modeles

| Critere | GPT-4o | Claude 3.7 | Mistral Large | Gemini |
|---|---|---|---|---|
| Raisonnement | 5/5 | 6/5 | 4/5 | 4/5 |
| Generation texte | 5/5 | 6/5 | 4/5 | 4/5 |
| Programmation | 5/5 | 4/5 | 4/5 | 4/5 |
| Analyse docs | 4/5 | 6/5 | 3/5 | 4/5 |
| Souverainete | Proprietaire | Proprietaire | Open source | Proprietaire |
| Ideal pour | Analyse strategique | Redaction pro | Deploiement local | Recherche documentaire |

---

## 5. Prompt Engineering

### Definition

Un **prompt** est l'ensemble des instructions fournies a un modele d'IA pour orienter la generation d'une reponse : role, objectif, contexte, format de sortie.

### Framework RISE

| Composante | Description |
|---|---|
| **R** — Role | Definir le role ou la persona du modele |
| **I** — Instructions | Formuler des instructions claires et precises |
| **S** — Steps | Decomposer en etapes logiques |
| **E** — Expected Output | Specifier le format, la longueur et la structure attendus |

### Structure d'un prompt avance (6 composantes)

1. **Role** : "Tu es un expert en..." — la fondation du prompt
2. **Contexte** : situation, secteur, donnees de depart
3. **Objectif** : ce que le modele doit produire concretement
4. **Etapes** : decomposition sequentielle de la tache
5. **Contraintes** : ton, longueur, sources, niveau de certitude
6. **Format de sortie** : tableau, JSON, plan structure, resume executif

### Techniques avancees de prompting

| Technique | Description | Cas d'usage |
|---|---|---|
| **Zero-shot** | Reponse directe, sans exemples | Taches simples et bien definies |
| **Few-shot** | 2 a 5 exemples dans le prompt | Classification, reformulation, contenu norme |
| **Chain of Thought** | Raisonnement etape par etape | Problemes complexes, estimation, decision |
| **Prompts multi-etapes** | Analyse + synthese + format structure | Livrables professionnels (rapports, recommandations) |

### Auto-verification anti-hallucination (3 phases)

1. **Redaction** : le modele redige une premiere reponse
2. **Critique** : il identifie les points incertains et risques d'erreur
3. **Amelioration** : reformulation finale avec niveau de confiance explicite

---

## 6. Assistants de code IA

| Outil | Editeur | Modele(s) | Forces |
|---|---|---|---|
| GitHub Copilot | VS Code, JetBrains | GPT-4o, Claude | Auto-completion, chat inline |
| Cursor | Fork de VS Code | Claude, GPT-4o | Edit multi-fichiers, composer |
| Claude Code | Terminal (CLI) | Claude | Autonomie maximale, projet complet |
| OpenAI Codex | ChatGPT (cloud) | o3, GPT-4o | Agent cloud, execution sandboxee |

**Regles** : toujours relire le code genere, tester avant integration, garder le controle humain sur l'architecture.

---

## 7. Appeler un LLM via l'API

### API REST (curl)

```bash
curl https://api.openai.com/v1/responses \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "gpt-4.1",
    "input": "Tu es un expert en support client B2B...",
    "temperature": 0.7,
    "top_p": 0.9,
    "max_output_tokens": 200
  }'
```

### SDK Python

```python
from openai import OpenAI

client = OpenAI()
response = client.responses.create(
    model="gpt-4.1",
    input="Tu es un consultant senior en strategie...",
    temperature=0.7,
    top_p=0.9,
    max_output_tokens=200,
)
print(response.output_text)
```

### Structured Outputs

Quand la reponse doit etre exploitee par une application, utiliser les Structured Outputs pour contraindre le format (JSON fiable) au niveau de l'API. Utile pour : interface web, CRM automatise, traitements enchaines, fiabilite garantie.

---

## 8. Parametres de generation

### Parametres cles

| Parametre | Role | Valeurs typiques |
|---|---|---|
| `temperature` | Controle la creativite | 0 = deterministe, 0.7 = equilibre, 1+ = creatif |
| `top_p` | Filtre les tokens par probabilite cumulee | 0.3 = conservateur, 1.0 = aucune restriction |
| `max_output_tokens` | Longueur maximale de la reponse | Selon besoin (1 token ≈ 3/4 mot FR) |
| `frequency_penalty` | Reduit les repetitions de mots | 0.1–0.5 recommande |
| `presence_penalty` | Encourage les nouveaux sujets | 0.1–0.5 recommande |

### Reglages par cas d'usage

| Cas d'usage | Temperature | Top-p | Profil |
|---|---|---|---|
| Analyse / resume de documents | 0.2 | 1.0 | Precis, factuel |
| Support client automatise | 0.3 | 1.0 | Stable, coherent |
| Generation de code | 0.1 | 1.0 | Deterministe, fiable |
| Redaction marketing | 0.8 | 1.0 | Creatif, engageant |
| Brainstorming / ideation | 0.9 | 1.0 | Explorateur, diversifie |

**Bonne pratique** : fixer `top_p = 1.0` et jouer uniquement sur `temperature` dans un premier temps.

---

## 9. Fiabilite et strategies anti-hallucination

4 strategies concretes :

1. **Prompts de validation et d'auto-critique** : demander au modele d'evaluer sa propre reponse
2. **Multi-reponses comparees** : generer plusieurs reponses independantes et comparer les convergences
3. **Ancrage sur documents de reference** : fournir des sources dans le prompt
4. **RAG** : connecter le LLM a une base documentaire externe en temps reel

---

## 10. Securite, donnees et conformite

### Risques identifies

- **Fuite de donnees sensibles** : donnees saisies pouvant etre utilisees pour l'entrainement
- **Propriete intellectuelle** : contenus generes pouvant reproduire des oeuvres protegees
- **Non-conformite RGPD** : traitement de donnees personnelles via des modeles heberges hors UE

### Bonnes pratiques

- **Anonymisation** : anonymiser/pseudonymiser les donnees avant soumission a un LLM externe
- **Politique de classification** : definir les niveaux de sensibilite et regles d'usage
- **Gouvernance IA** : charte d'utilisation, roles de supervision, processus de revue reguliers

---

## 11. Personnalisation des modeles

3 approches par ordre de complexite croissante :

| Approche | Description | Quand l'utiliser |
|---|---|---|
| **1. Prompt Engineering** | Rapide, flexible, sans infrastructure | Couvre 80% des besoins — **commencer toujours par la** |
| **2. RAG** | Connexion a des documents internes, reponses ancrees et tracables | Bases de connaissances, documentation metier, FAQ |
| **3. Fine-tuning** | Specialisation profonde du modele sur un corpus proprietaire | Fort volume, haute valeur ajoutee, cout eleve |

> **Regle d'or** : commencez TOUJOURS par le prompt engineering. 80% des cas d'usage en entreprise se resolvent avec un bon prompt.

---

## Competences acquises

- Comprendre le fonctionnement reel des LLM et anticiper leurs comportements
- Choisir le bon modele selon le cas d'usage, le cout et la souverainete
- Rediger des prompts avances (framework RISE)
- Fiabiliser les reponses (strategies anti-hallucination, RAG)
- Securiser l'usage (RGPD, anonymisation, gouvernance IA)
- Integrer l'IA dans ses processus (prompting, RAG, fine-tuning)
