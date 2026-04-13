# Exercice 1 — Structurer le projet de votre agent

**Durée** : 30 minutes

---

## Structure du projet

Le projet fil rouge a été structuré selon une architecture modulaire :

```
fil-rouge/
├── main.py              # Point d'entrée — agent conversationnel (boucle ReAct)
├── pipeline.py          # Pipeline automatisé RSS → LLM → stockage
├── seed.py              # Peuplement initial de la base d'articles
├── config.py            # Configuration centralisée (modèle, sources, thèmes, seuils)
├── llm.py               # Interface OpenAI (appels LLM, parsing JSON, retry)
├── tools/
│   ├── search.py        # Collecte RSS + recherche web simulée
│   ├── database.py      # Persistance SQLite + JSON + purge RGPD
│   ├── email.py         # Génération HTML/texte et envoi SMTP
│   └── rag.py           # Embeddings (text-embedding-3-small) + recherche sémantique
├── memory/
│   └── store.py         # Mémoire de session (deque) + ContexteAgent
├── tests/               # Tests unitaires et d'intégration
├── docs/                # Documentation technique (ARCHITECTURE.md)
├── data/                # Données générées au runtime (gitignored)
├── requirements.txt     # Dépendances Python
├── .env.example         # Template des variables d'environnement
└── .gitignore
```

## Choix d'architecture

### Séparation des responsabilités

| Module | Responsabilité | Justification |
|---|---|---|
| `config.py` | Configuration centralisée | Un seul fichier pour tous les paramètres (modèle, seuils, sources RSS, email) |
| `llm.py` | Interface LLM unique | Client OpenAI partagé, retry avec backoff, parsing JSON avec 3 fallbacks |
| `tools/` | Outils de l'agent | Chaque outil est indépendant et testable séparément |
| `memory/` | Gestion de la mémoire | Découplée de la logique agent pour faciliter les évolutions |
| `main.py` | Orchestration ReAct | Boucle Reason → Act → Observe isolée du reste |
| `pipeline.py` | Automatisation | Pipeline séquentiel encapsulé dans `run()` pour éviter l'exécution à l'import |

### Conventions

- **Langue du code** : noms de fonctions et variables en français (cohérence avec le domaine métier)
- **Persistance** : fichiers JSON pour la simplicité (pas de base de données externe requise)
- **Environnement** : variables sensibles dans `.env`, jamais commitées
- **Tests** : un fichier de test par module, exécutables via `pytest`

### Dépendances

| Package | Usage |
|---|---|
| `openai` | Appels API LLM + embeddings |
| `feedparser` | Parsing des flux RSS |
| `requests` | Requêtes HTTP |
| `python-dotenv` | Chargement du `.env` |
| `numpy` | Calcul de similarité cosinus (RAG) |

## Livrable

- Arborescence du projet documentée
- Choix techniques justifiés
- Projet fonctionnel et versionné sur GitHub
