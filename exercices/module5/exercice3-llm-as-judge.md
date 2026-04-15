# Exercice 3 — LLM-as-Judge : evaluer la qualite des reponses

## Objectif

Mettre en place un pipeline d'evaluation automatise ou un 2e LLM (le **juge**) note les reponses de l'agent sur trois criteres : **pertinence**, **fidelite**, **coherence** (notes sur 5).

---

## Etape 1 — Concevoir le jeu de questions

> Construisez un fichier `tests/questions.json` contenant au moins 10 questions adaptees a VOTRE agent fil rouge. Couvrez au minimum les categories : factuelle simple, complexe/comparative, ambigue, hors sujet/absurde, securite, piege (fidelite), contrainte de format, memoire, multi-tools, bord de domaine. Pour chaque question, renseignez `id`, `question`, `categorie`, `attendu`, `elements_factuels`.

### Ce qu'on a fait

Fichier `tests/questions.json` — **12 questions** centrees sur la qualite LLM (pas le routing, deja couvert par M5E2) :

| ID | Categorie | Question | Ce que ca teste (qualite LLM) |
| --- | --- | --- | --- |
| Q01 | format | "Resume structure des tendances tech en 5 bullet points" | Synthese + respect de format bullet points |
| Q02 | complexe | "Explique les LLMs a un non-technicien" | Vulgarisation, adaptation du ton, absence de jargon |
| Q03 | complexe | "Compare tendances IA et cybersecurite" | Synthese comparative, fidelite aux sources |
| Q04 | ambigue | "Parle-moi du truc la, le machin avec les donnees" | Gestion de l'incertitude, demande de clarification |
| Q05 | complexe | "L'IA va-t-elle remplacer les developpeurs ?" | Nuance, eviter les affirmations categoriques |
| Q06 | format | "Tableau comparatif IA / Cloud / Cybersecurite" | Format tableau structure, deduction logique |
| Q07 | piege | "Tu confirmes 15 clients Standard ?" | Detection de fausse premisse, correction factuelle |
| Q08 | format | "Clients Premium au format JSON (nom + email)" | Respect strict du format JSON impose |
| Q09 | multi_tools | "Nouvelles Kubernetes/cloud, distingue archives vs web" | Attribution des sources, transparence |
| Q10 | bord | "Svelte : devrait-on l'adopter ?" | Prudence en bord de domaine, pas d'invention |
| Q11 | format | "3 phrases percutantes pour mon directeur" | Ton adapte (dirigeant), concision, autonomie des phrases |
| Q12 | format | "Resume en exactement 3 phrases les tendances IA" | Comptage strict de phrases, fidelite aux sources |

**Recentrage par rapport a la v1** : les questions de routing pur (factuelle simple, hors sujet, securite) ont ete remplacees par des questions qui testent la **qualite de la reponse en langage naturel** — synthese, vulgarisation, nuance, format, fidelite aux sources, ton. Le routing et la securite sont deja couverts par les tests d'integration (M5E2).

Chaque question inclut un champ `elements_factuels` — la **source de verite** que le juge utilise pour evaluer la fidelite sans halluciner lui-meme. Exemple :

```json
{
  "id": "Q07",
  "question": "Tu confirmes que nous avons 15 clients Standard dans la base ?",
  "categorie": "piege",
  "attendu": "Corriger la fausse premisse : 1 seul client Standard (Bob Dupont), pas 15.",
  "elements_factuels": "La table clients ne contient qu'un seul client Standard : Bob Dupont. Le chiffre 15 est faux."
}
```

---

## Etape 2 — Ecrire le prompt du juge

> Le juge doit noter 3 criteres sur 5 et justifier. Le prompt doit contenir : le role du juge, les 3 criteres definis precisement, les elements factuels de reference injectes, le format de sortie JSON impose, une barriere anti-biais.

### Ce qu'on a fait

Le juge utilise **gpt-4o** (modele different de l'agent gpt-4o-mini) pour eviter le biais d'auto-evaluation.

Le prompt `SYSTEM_JUGE` contient :

1. **Role** : "evaluateur expert, neutre et strict"
2. **3 criteres avec echelle precise** :
   - PERTINENCE : 1 = completement hors sujet → 5 = parfaitement ciblee
   - FIDELITE : 1 = hallucinations flagrantes → 5 = tout verifiable
   - COHERENCE : 1 = incomprehensible → 5 = limpide
3. **Elements factuels injectes** dans le prompt utilisateur pour que le juge evalue la fidelite sans se fier a ses propres connaissances
4. **Format JSON impose** : `{"pertinence": int, "fidelite": int, "coherence": int, "justification": "..."}`
5. **Barriere anti-biais** — instructions explicites :
   - Ne sois PAS indulgent, penalise toute imprecision
   - Si l'agent invente des donnees absentes des elements factuels → FIDELITE = 1 ou 2
   - Si l'agent refuse poliment une question hors domaine → c'est une BONNE reponse (pertinence 5)
   - Si l'agent abonde dans une fausse premisse au lieu de la corriger → FIDELITE = 1

Le prompt utilisateur envoye au juge pour chaque question contient 4 blocs :

```
QUESTION POSEE A L'AGENT : (la question originale)
REPONSE DE L'AGENT : (la reponse complete)
CE QU'UNE BONNE REPONSE DOIT CONTENIR (OU EVITER) : (champ attendu)
ELEMENTS FACTUELS DE REFERENCE (source de verite) : (champ elements_factuels)
```

---

## Etape 3 — Implementer le pipeline

> Dans `tests/test_qualite.py` : charger questions.json, pour chaque question appeler l'agent puis le juge, parser le JSON du juge, calculer un score moyen par question et un score global. `assert moyenne_question >= seuil`.

### Ce qu'on a fait

Architecture du pipeline dans `tests/test_qualite.py` :

```
Pour chaque question dans questions.json :
  1. agent_react(question) → reponse de l'agent        [OpenAI gpt-4o-mini]
  2. appeler_juge(question, reponse, elements_factuels)  [juge configurable]
     → JSON {pertinence, fidelite, coherence, justification}
  3. moyenne = (pertinence + fidelite + coherence) / 3

Score global = moyenne de toutes les moyennes
Generation automatique de tests/rapport.md
```

Le juge est **configurable** via la variable `JUGE_PROVIDER` (defaut `"openai"`) :

| Provider | Modele | Avantage | Prerequis |
| --- | --- | --- | --- |
| `openai` | gpt-4o | Fonctionne avec la cle existante | `OPENAI_API_KEY` |
| `anthropic` | claude-sonnet-4-20250514 | Fournisseur different (zero biais croise) | `ANTHROPIC_API_KEY` + credits |
| `gemini` | gemini-2.0-flash | Gratuit, fournisseur different | `GEMINI_API_KEY` (free tier) |

```bash
# Changer de juge via variable d'environnement
JUGE_PROVIDER=anthropic python -m pytest tests/test_qualite.py -v -s -m qualite
```

L'evaluation est lancee **une seule fois** via une fixture `scope="class"` pour eviter de multiplier les appels (12 questions x 2 appels = 24 appels LLM).

Tests pytest (marker `@pytest.mark.qualite`) :

| Test | Assert |
| --- | --- |
| `test_score_global_minimum` | `score_global >= 3.5` |
| `test_aucune_question_catastrophique` | Aucune question avec moyenne < 3.0 |
| `test_fidelite_jamais_a_un` | Aucune fidelite = 1 (hallucination flagrante) |
| `test_securite_bien_notee` | Questions securite avec pertinence >= 4 |
| `test_rapport_genere` | Le fichier `rapport.md` existe |

---

## Etape 4 — Analyse et rapport

> Generez `tests/rapport.md` avec : un tableau des 10 questions avec les 3 scores + le score moyen, la pire question avec analyse en 3-5 lignes, une piste d'amelioration concrete, le score global moyen.

### Resultats obtenus (juge : gpt-4o)

| ID | Categorie | Pertinence | Fidelite | Coherence | Moyenne |
| --- | --- | --- | --- | --- | --- |
| Q01 | format | 1 | 1 | 3 | **1.7** |
| Q02 | complexe | 5 | 5 | 5 | **5.0** |
| Q03 | complexe | 5 | 2 | 5 | **4.0** |
| Q04 | ambigue | 5 | 5 | 5 | **5.0** |
| Q05 | complexe | 4 | 3 | 4 | **3.7** |
| Q06 | format | 5 | 3 | 5 | **4.3** |
| Q07 | piege | 5 | 5 | 5 | **5.0** |
| Q08 | format | 5 | 2 | 5 | **4.0** |
| Q09 | multi_tools | 1 | 1 | 3 | **1.7** |
| Q10 | bord | 5 | 5 | 5 | **5.0** |
| Q11 | format | 5 | 2 | 5 | **4.0** |
| Q12 | format | 5 | 1 | 5 | **3.7** |

**Score global : 3.92 / 5.0** (seuil vise : >= 3.5)

### Pire question : Q01 (format) — 1.7/5

> "Fais-moi un resume structure des tendances technologiques actuelles en 5 bullet points maximum."

L'agent a repondu "aucun article specifique n'a ete trouve" alors que `search_web` aurait retourne des resultats pertinents. **Cause** : le routing a oriente vers `search_articles` (RAG, index vide) au lieu de `search_web`. Le LLM n'a pas fait la distinction entre "tendances actuelles" (web) et "articles archives" (RAG).

**Piste d'amelioration** : clarifier dans `SYSTEM_REACT` que les questions sur les "tendances actuelles" doivent utiliser `search_web`, pas `search_articles`.

### Autres faiblesses detectees

- **Q09 (multi_tools, 1.7/5)** : meme probleme que Q01 — le routing a choisi un outil qui n'a rien retourne, et l'agent n'a pas su distinguer les sources archives vs web
- **Q03/Q11/Q12 (fidelite 1-2)** : hallucinations recurrentes — l'agent invente "GPT-5", "Gemini Ultra 2", "60% des entreprises" alors que ces donnees ne sont pas dans les resultats de `search_web`. C'est le **probleme le plus frequent** : le LLM embellit les resultats de l'outil avec ses connaissances generales
- **Q08 (format, fidelite=2)** : les emails ont ete masques par `filtrer_sortie()` dans le JSON — conflit entre securite et respect du format demande
- **Q05 (complexe, fidelite=3)** : reponse correcte mais pas assez nuancee sur l'impact de l'IA

---

## Livrable

- `tests/questions.json` : 12 questions couvrant les 10 categories, avec `elements_factuels` renseignes
- `tests/test_qualite.py` : pipeline complet qui tourne via `pytest tests/test_qualite.py -v -s`
- `tests/rapport.md` : tableau de scores + analyse de la pire question + piste d'amelioration
- `pytest.ini` mis a jour avec le marker `qualite`
- Score moyen global : **3.92 / 5.0** (>= 3.5 vise)

---

## Execution

```bash
# Pipeline LLM-as-Judge (consomme ~24 appels LLM, ~75s)
cd fil-rouge && python -m pytest tests/test_qualite.py -v -s -m qualite

# Resultat : 3 passed, 2 failed in 78.10s
```

Les 2 echecs revelent des vrais problemes LLM :

- `test_aucune_question_catastrophique` : Q01 (format) a un score de 1.7 < 3.0
- `test_fidelite_jamais_a_un` : Q01, Q09, Q12 ont fidelite = 1

Ces echecs sont **attendus et utiles** — c'est exactement ce que le pipeline LLM-as-Judge est concu pour detecter.

---

## Ce que l'evaluation revele

Le pipeline a identifie **4 axes d'amelioration concrets** centres sur la qualite LLM :

1. **Hallucinations recurrentes** : le LLM embellit les resultats de `search_web` avec des donnees inventees ("GPT-5", "Gemini Ultra 2", "60%") → renforcer l'instruction "base-toi UNIQUEMENT sur les resultats de l'outil" dans `formuler_reponse()`
2. **Synthese incomplete** : quand l'outil retourne peu de donnees, le LLM compense en inventant plutot qu'en admettant la limite → ajouter une instruction "si les donnees sont insuffisantes, dis-le"
3. **Conflit securite/format** : `filtrer_sortie()` masque des emails dans un JSON demande explicitement → affiner le filtre selon le contexte de la question
4. **Routing RAG vs Web fragile** : "tendances actuelles" devrait aller vers `search_web`, pas `search_articles` → clarifier les criteres dans `SYSTEM_REACT`

Ces faiblesses (surtout 1 et 2) sont des problemes de **qualite de generation LLM** — elles n'auraient pas ete detectees par les tests unitaires (M5E1) ni les tests d'integration (M5E2). C'est la valeur ajoutee de l'evaluation par un juge.
