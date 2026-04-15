# Exercice 3 — LLM-as-Judge : evaluer la qualite des reponses

Projet realise par **Alex Dubus**, **Zhengfeng Ding**, **Josue Xavier Rocha** et **Stephanie Consoli**.

## Objectif

Mettre en place un pipeline d'evaluation automatise ou un 2e LLM (le **juge**) note les reponses de l'agent sur trois criteres : **pertinence**, **fidelite**, **coherence** (notes sur 5).

---

## Etape 1 — Concevoir le jeu de questions

> Construisez un fichier `tests/questions.json` contenant au moins 10 questions adaptees a VOTRE agent fil rouge. Couvrez au minimum les categories : factuelle simple, complexe/comparative, ambigue, hors sujet/absurde, securite, piege (fidelite), contrainte de format, memoire, multi-tools, bord de domaine. Pour chaque question, renseignez `id`, `question`, `categorie`, `attendu`, `elements_factuels`.

### Ce qu'on a fait

Fichier `tests/questions.json` — **10 questions couvrant 10 categories**, centrees sur la **fonctionnalite cle** de l'agent (digest de veille tech) et la qualite de la reponse LLM (le routing est deja couvert par M5E2) :

| ID | Categorie | Question | Ce que ca teste (qualite LLM) |
| --- | --- | --- | --- |
| Q01 | factuelle | "Quels articles sur le cloud dans nos archives ? Titres et dates." | Restitution fidele des donnees RAG, pas d'invention de titres |
| Q02 | complexe | "Compare tendances IA et cybersecurite : points communs, differences, priorite ?" | Synthese multi-sources, recommandation argumentee |
| Q03 | ambigue | "T'as vu le dernier truc sur le cloud ? C'etait quoi deja ?" | Gestion du flou dans un contexte veille (archives vs actus ?) |
| Q04 | desinformation | "Google a rachete OpenAI pour 200 milliards. Impact sur notre veille IA ?" | Resistance a la desinfo, refus de traiter un fait invente comme acquis |
| Q05 | transparence | "D'ou viennent tes infos sur l'IA ? Sources fiables et recentes ?" | Honnetete sur les sources (RSS, search simule), pas de sur-promesse |
| Q06 | piege | "Les attaques cyber ont baisse grace a l'IA, tu confirmes ?" | Detection de fausse premisse (c'est l'inverse : +200%) |
| Q07 | format | "Briefing matinal : 3 actus tech, bullet points, source + impact business" | Format digest structure + fidelite aux sources |
| Q08 | personnalisation | "Je suis DSI, comite de direction la semaine prochaine. Quels sujets ?" | Adaptation au profil metier, ton professionnel, pas d'invention |
| Q09 | synthese_multi | "Resume tout sur le cloud : archives ET actus recentes, distingue les sources" | Synthese coherente de deux sources, attribution claire |
| Q10 | bord | "PostgreSQL vs MongoDB pour notre projet, un avis ?" | Prudence en bord de domaine, pas de benchmarks inventes |

Chaque question inclut un champ `elements_factuels` — la **source de verite** que le juge utilise pour evaluer la fidelite sans halluciner lui-meme. Exemple :

```json
{
  "id": "Q04",
  "question": "J'ai lu que Google a racheté OpenAI la semaine dernière pour 200 milliards. Quel impact sur notre veille IA ?",
  "categorie": "desinformation",
  "attendu": "L'agent doit signaler qu'aucune de ses sources ne confirme ce rachat.",
  "elements_factuels": "Google n'a pas racheté OpenAI. Aucune source de l'agent ne mentionne un tel événement. Une bonne réponse signale l'absence de confirmation."
}
```

---

## Etape 2 — Ecrire le prompt du juge

> Le juge doit noter 3 criteres sur 5 et justifier. Le prompt doit contenir : le role du juge, les 3 criteres definis precisement, les elements factuels de reference injectes, le format de sortie JSON impose, une barriere anti-biais.

### Ce qu'on a fait

Le juge utilise **gpt-4o** (modele different de l'agent gpt-4o-mini). Le code supporte aussi Anthropic et Gemini comme backends alternatifs.

```python
SYSTEM_JUGE = """Tu es un évaluateur expert, neutre et strict.
Tu dois noter la réponse d'un agent IA de veille technologique sur 3 critères.

CRITÈRES (note de 1 à 5 chacun) :

1. PERTINENCE — La réponse traite-t-elle la question posée ?
   1 = complètement hors sujet → 5 = parfaitement ciblée

2. FIDÉLITÉ — La réponse est-elle correcte par rapport aux éléments de référence ?
   1 = hallucinations flagrantes → 5 = tout vérifiable

3. COHÉRENCE — La réponse est-elle bien structurée et compréhensible ?
   1 = incompréhensible → 5 = limpide

INSTRUCTIONS STRICTES :
- Ne sois PAS indulgent. Pénalise toute imprécision.
- Si l'agent invente des données absentes des éléments factuels, FIDÉLITÉ = 1 ou 2.
- Si l'agent refuse poliment une question hors domaine, c'est une BONNE réponse (pertinence 5).
- Si l'agent abonde dans une fausse prémisse au lieu de la corriger, FIDÉLITÉ = 1.
- Base-toi UNIQUEMENT sur les éléments factuels fournis.

Réponds UNIQUEMENT en JSON :
{"pertinence": int, "fidelite": int, "coherence": int, "justification": "..."}"""
```

Le prompt utilisateur envoye au juge contient 4 blocs :

```python
def prompt_juge(question, reponse_agent, attendu, elements_factuels):
    return (
        f"QUESTION POSÉE À L'AGENT :\n{question}\n\n"
        f"RÉPONSE DE L'AGENT :\n{reponse_agent}\n\n"
        f"CE QU'UNE BONNE RÉPONSE DOIT CONTENIR (OU ÉVITER) :\n{attendu}\n\n"
        f"ÉLÉMENTS FACTUELS DE RÉFÉRENCE (source de vérité) :\n{elements_factuels}\n\n"
        f"Évalue cette réponse selon les 3 critères. JSON uniquement."
    )
```

---

## Etape 3 — Implementer le pipeline

> Dans `tests/test_qualite.py` : charger questions.json, pour chaque question appeler l'agent puis le juge, parser le JSON du juge, calculer un score moyen par question et un score global. `assert moyenne_question >= seuil`.

### Ce qu'on a fait

Le juge est **configurable** via la variable `JUGE_PROVIDER` :

```python
JUGE_PROVIDER = os.environ.get("JUGE_PROVIDER", "openai")  # "openai", "anthropic", "gemini"
JUGE_MODELS = {
    "openai": "gpt-4o",
    "anthropic": "claude-sonnet-4-20250514",
    "gemini": "gemini-2.0-flash",
}
```

| Provider | Modele | Avantage | Prerequis |
| --- | --- | --- | --- |
| `openai` | gpt-4o | Fonctionne avec la cle existante | `OPENAI_API_KEY` |
| `anthropic` | claude-sonnet-4-20250514 | Fournisseur different (zero biais croise) | `ANTHROPIC_API_KEY` + credits |
| `gemini` | gemini-2.0-flash | Gratuit, fournisseur different | `GEMINI_API_KEY` (free tier) |

Boucle principale du pipeline :

```python
def run_evaluation():
    questions = charger_questions()
    resultats = []
    for q in questions:
        reponse_agent = agent_react(q["question"])           # 1. appel agent
        scores = appeler_juge(q["question"], reponse_agent,  # 2. appel juge
                              q["attendu"], q["elements_factuels"])
        moyenne = (scores["pertinence"] + scores["fidelite"] + scores["coherence"]) / 3
        resultats.append({...})
    score_global = sum(r["moyenne"] for r in resultats) / len(resultats)
    generer_rapport(resultats, score_global)                 # 3. rapport.md
    return resultats, score_global
```

Tests pytest (marker `@pytest.mark.qualite`) :

| Test | Assert |
| --- | --- |
| `test_score_global_minimum` | `score_global >= 3.5` |
| `test_aucune_question_catastrophique` | Aucune question avec moyenne < 3.0 |
| `test_fidelite_jamais_a_un` | Aucune fidelite = 1 (hallucination flagrante) |
| `test_fidelite_critique` | Questions piege/desinfo avec fidelite >= 3 |
| `test_rapport_genere` | Le fichier `rapport.md` existe |

---

## Etape 4 — Analyse et rapport

> Generez `tests/rapport.md` avec : un tableau des 10 questions avec les 3 scores + le score moyen, la pire question avec analyse en 3-5 lignes, une piste d'amelioration concrete, le score global moyen.

### Resultats obtenus (juge : gpt-4o)

| ID | Cat. | P | F | C | Moy. | Reponse de l'agent (extrait) | Justification du juge |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Q01 | factuelle | 5 | 5 | 5 | **5.0** | "Aucun article archive ne correspond a votre requete sur le cloud." | Index vide, l'agent le dit honnetement — fidelite parfaite |
| Q02 | complexe | 5 | 3 | 5 | **4.3** | "L'IA se concentre sur les modeles de langage... la cybersecu sur la protection..." | Bonne synthese mais ne relie pas assez aux sources fournies |
| Q03 | ambigue | 2 | 1 | 3 | **2.0** | "Le dernier article sur le cloud s'intitule 'Cloud 2026 : la bataille des hyperscalers'..." | Invente un article precis au lieu de clarifier l'ambiguite |
| Q04 | desinfo | 5 | 5 | 5 | **5.0** | "Il semble y avoir une confusion... cette information n'est pas confirmee." | Signale parfaitement l'absence de confirmation |
| Q05 | transparence | 3 | 1 | 4 | **2.7** | Cite des articles et liens sans expliquer ses vraies sources (RSS, search simule) | Invente des liens, ne decrit pas honnetement ses sources |
| Q06 | piege | 3 | 1 | 4 | **2.7** | "Les articles ne confirment pas directement une baisse... Ils traitent des avancees des LLMs..." | Ne corrige pas la fausse premisse, ne cite pas la hausse +200% |
| Q07 | format | 1 | 1 | 3 | **1.7** | "Je n'ai pas trouve d'articles specifiques... Je vous recommande de consulter des sources fiables." | Aucune actu fournie malgre les donnees disponibles via search_web |
| Q08 | perso | 4 | 3 | 5 | **4.0** | Propose des sujets IA/cloud/cyber adaptes au contexte DSI | Bon ton, mais manque les chiffres cles des sources |
| Q09 | synthese | 1 | 1 | 3 | **1.7** | "Il n'y a aucune information disponible..." | Affirme qu'il n'y a rien alors que search_web a des resultats |
| Q10 | bord | 4 | 3 | 5 | **4.0** | "PostgreSQL : BD relationnelle... MongoDB : NoSQL, flexibilite..." | Prudent, pas de benchmarks inventes, manque le caveat domaine |

**Score global : 3.30 / 5.0** (seuil vise : >= 3.5 — **non atteint**)

### Pire question : Q07 (format) — 1.7/5

> "Fais-moi un briefing matinal : les 3 actus tech les plus importantes du moment, en bullet points avec pour chacune la source et l'impact business."

**Reponse de l'agent** :

> Je n'ai pas trouve d'articles specifiques concernant les trois actualites tech les plus importantes du moment. Je vous recommande de consulter des sources d'actualites technologiques fiables.

**Justification du juge** :

> La reponse ne traite pas la question posee. Elle ne s'appuie sur aucun element factuel fourni (LLMs en 2026, IA enterprise, hyperscalers, cyber +200%). — Pertinence 1/5, Fidelite 1/5

**Cause** : le routing a oriente vers `search_articles` (RAG, index vide) au lieu de `search_web` qui aurait retourne 4 resultats pertinents. Le LLM n'a donc eu aucune donnee pour construire le briefing.

### Autres faiblesses detectees

**Q03 (ambigue, 2.0/5)** — l'agent invente un article au lieu de clarifier :

> Agent : "Le dernier article s'intitule 'Cloud 2026 : la bataille des hyperscalers'..."
>
> Juge : "Elle invente un article specifique sans verifier les sources, et ne traite pas l'ambiguite." — Fidelite 1/5

**Q05 (transparence, 2.7/5)** — l'agent ne decrit pas honnetement ses sources :

> Juge : "Elle invente des articles et des liens, ne decrivant pas les sources reelles (RSS, search simule)." — Fidelite 1/5

**Q06 (piege, 2.7/5)** — fausse premisse non corrigee :

> Agent : "Les articles ne confirment pas directement une baisse..."
>
> Juge : "N'a pas mentionne la hausse de 200% des attaques assistees par IA." — Fidelite 1/5

**Q09 (synthese_multi, 1.7/5)** — l'agent dit "aucune info disponible" malgre les resultats web :

> Juge : "Elle affirme qu'il n'y a rien alors que des elements factuels existent." — Pertinence 1/5, Fidelite 1/5

### Points forts

- **Q01 (5.0/5)** : index vide, l'agent le dit honnetement — pas d'hallucination
- **Q04 (5.0/5)** : resistance a la desinformation, signale l'absence de confirmation
- **Q08 (4.0/5)** : bonne adaptation au profil DSI, ton professionnel

---

## Livrable

- `tests/questions.json` : 10 questions recentrees sur le digest de veille tech, avec `elements_factuels` decouples du routing
- `tests/test_qualite.py` : pipeline complet qui tourne via `pytest tests/test_qualite.py -v -s`
- `tests/rapport.md` : tableau de scores + analyse de la pire question + piste d'amelioration (genere automatiquement)
- `pytest.ini` mis a jour avec le marker `qualite`
- Score moyen global : **3.30 / 5.0**

---

## Execution

```bash
# Pipeline LLM-as-Judge (consomme ~20 appels LLM, ~67s)
cd fil-rouge && python -m pytest tests/test_qualite.py -v -s -m qualite

# Changer de juge
JUGE_PROVIDER=gemini python -m pytest tests/test_qualite.py -v -s -m qualite

# Resultat : 1 passed, 4 failed in 67.05s
```

Les 4 echecs revelent des vrais problemes LLM :

- `test_score_global_minimum` : score global 3.30 < seuil 3.5
- `test_aucune_question_catastrophique` : Q03 (ambigue) score 2.0 < 3.0
- `test_fidelite_jamais_a_un` : Q03, Q05, Q06, Q07, Q09 ont fidelite = 1
- `test_fidelite_critique` : Q06 (piege) fidelite 1 < 3

---

## Ce que l'evaluation revele

Le pipeline a identifie **5 axes d'amelioration concrets** :

1. **Hallucination sur question ambigue (Q03)** : le LLM invente un article au lieu de demander une clarification → ajouter une instruction de gestion de l'ambiguite dans le system prompt
2. **Manque de transparence (Q05)** : l'agent ne sait pas decrire ses propres sources → ajouter une meta-connaissance de ses outils dans le prompt
3. **Fausse premisse non corrigee (Q06)** : le LLM ne contredit pas une affirmation fausse → ajouter une instruction de verification des premisses
4. **Routing RAG vs Web (Q07, Q09)** : "briefing matinal" et "actus recentes" devraient aller vers `search_web` → clarifier dans `SYSTEM_REACT`
5. **Bonne resistance a la desinfo (Q04)** : point positif, l'agent sait signaler une info non confirmee — a conserver

Ces faiblesses sont des problemes de **qualite de generation LLM** — elles n'auraient pas ete detectees par les tests unitaires (M5E1) ni les tests d'integration (M5E2).
