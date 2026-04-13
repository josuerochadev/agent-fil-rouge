# Exercice 1 — Construire un pipeline RAG de bout en bout

**Durée** : 45 minutes
**Matériel** : Python, `pip install chromadb openai pdfplumber`

---

## Étape 1 — Ingestion et nettoyage

Fichier : [`rag/ingest.py`](rag/ingest.py)

- Extraction du texte de chaque page PDF avec `pdfplumber`
- Nettoyage des espaces multiples et sauts de ligne superflus
- Métadonnées conservées : source (nom du fichier), numéro de page

**Résultat** : 39 PDFs CNIL ingérés, **1749 pages** extraites.

## Étape 2 — Chunking

Fichier : [`rag/chunk.py`](rag/chunk.py)

- Découpage en morceaux de ~500 mots avec chevauchement de 50 mots
- Chaque chunk conserve les métadonnées de sa page source

**Résultat** : 1749 pages → **2367 chunks**.

## Étape 3 — Embeddings + Stockage

Fichier : [`rag/vectorstore.py`](rag/vectorstore.py)

- Embeddings générés avec `text-embedding-3-small` (OpenAI) en batch de 100
- Stockage dans ChromaDB (persistant sur disque) avec métadonnées (source, page)
- Métrique de distance : cosinus

**Résultat** : **2367 chunks** indexés dans ChromaDB.

## Étape 4 — Interrogation

Fichier : [`rag/query.py`](rag/query.py)

- Recherche des 3 chunks les plus pertinents par similarité cosinus
- Injection du contexte dans le prompt avec l'instruction "réponds uniquement à partir du contexte fourni"
- Génération de la réponse via `gpt-4o-mini` (température 0.2)

## Étape 5 — Tests

| Question | Réponse pertinente ? | Source citée ? | Hors corpus géré ? |
|---|---|---|---|
| Quels sont les droits des personnes concernées selon le RGPD ? | Oui — liste complète des 6 droits | Oui — referentiel_protection_enfance.pdf (p.21), guide_open_data.pdf (p.16) | — |
| Quelles sont les recommandations de la CNIL en matière de sécurité des données ? | Oui — recommandations détaillées (sensibilisation, mesures, logiciels) | Oui — cnil_plaquette_cybersecurite_2025.pdf (p.4), projet_de_recommandation_dossier_patient_informatise.pdf (p.4) | — |
| Quel est le cours actuel de l'action Apple en bourse ? | — | — | Oui — "Je ne dispose pas d'informations..." (scores ~0.27) |

Les scores de similarité confirment la pertinence : **0.72-0.76** pour les questions dans le corpus, **~0.27** pour la question hors corpus.

## Livrable

- Pipeline RAG fonctionnel : 4 fichiers Python dans [`rag/`](rag/)
- Corpus indexé : 39 PDFs CNIL, 2367 chunks
- 3 tests documentés ci-dessus
