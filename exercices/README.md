# Exercices

Exercices pratiques réalisés pendant la formation IA, organisés par module.

## Module 4 — RAG (Retrieval-Augmented Generation)

### Exercice 1 — Pipeline RAG de bout en bout

Pipeline complet de question-réponse sur un corpus de documents CNIL (RGPD, sécurité des données).

```
module4/rag/
├── ingest.py        # Extraction et nettoyage des PDF
├── chunk.py         # Découpage en chunks (~500 mots, chevauchement 50)
├── vectorstore.py   # Embeddings (text-embedding-3-small) + stockage ChromaDB
└── query.py         # Interrogation RAG + génération de réponse LLM
```

**Corpus** : ~40 guides et rapports CNIL (1749 pages, 2367 chunks indexés).

**Résultats des tests** :

| Question | Pertinente ? | Source citée ? | Hors corpus géré ? |
|---|---|---|---|
| Droits des personnes selon le RGPD | Oui | Oui | — |
| Recommandations sécurité CNIL | Oui | Oui | — |
| Cours de l'action Apple | — | — | Oui |
