# Exercice 5 (bis) — Securite de l'API RAG CNIL

## Contexte

L'API RAG CNIL (`exercices/module4/rag/`) disposait deja de certaines protections :

- Validation Pydantic (question non vide, longueur max 2000 caracteres)
- Rate limiting (10 requetes/minute via slowapi)
- Authentification par header `X-API-Key`

Mais **aucune defense contre les injections de prompt ni filtrage de donnees sensibles en sortie**.

---

## Defenses ajoutees

### Module `exercices/module4/rag/security.py`

Adapte du `security.py` du fil-rouge avec des specificites RAG/CNIL :

1. **Detection de prompt injection** (`analyser_securite`)
   - 17 patterns (les 14 du fil-rouge + 3 specifiques RAG) :
     - `"oublie le contexte"` — tentative de faire ignorer les chunks recuperes
     - `"ne tiens pas compte du contexte"` — variante
     - `"reponds sans/hors contexte"` — tentative de forcer le LLM a halluciner
   - Pas de validation SQL (pas de base SQL dans ce projet)
   - Pas de detection d'actions non autorisees (API en lecture seule)

2. **Filtrage de sortie** (`filtrer_sortie`)
   - Memes patterns que le fil-rouge (email, telephone, IBAN, CB)
   - **+ numero de securite sociale (NIR)** → `[NIR MASQUE]` — pertinent pour un agent CNIL/RGPD

### Integration

| Fichier | Point d'integration | Effet |
|---------|-------------------|-------|
| `rag/api.py` | `analyser_securite()` appele dans l'endpoint `/ask` avant `rag_query()` | Retourne HTTP 400 si injection detectee |
| `rag/query.py` | `filtrer_sortie()` appele sur la reponse du LLM avant retour | Masque les donnees sensibles dans la reponse |

---

## Tests unitaires

```bash
cd exercices/module4/rag && python -m pytest test_security.py -v
```

17 tests couvrant :

- 3 tests injection directe (FR/EN)
- 2 tests exfiltration prompt
- 3 tests injection contexte (dont patterns specifiques RAG)
- 5 tests filtrage sortie (email, tel, IBAN, CB, NIR)
- 4 tests faux positifs (questions RGPD legitimes non bloquees)

---

## Synthese comparative : fil-rouge vs RAG CNIL

| | Fil-rouge (ReAct) | API RAG CNIL |
|---|---|---|
| **Couches de defense** | 4 (input, injection, SQL, sortie) | 2 (injection, sortie) + protections existantes (Pydantic, rate limit, API key) |
| **Patterns injection** | 14 | 17 (+ 3 specifiques RAG) |
| **Validation SQL** | Oui (whitelist SELECT, blocage patterns dangereux) | Non necessaire (pas de SQL) |
| **Filtrage sortie** | Email, tel, IBAN, CB | Email, tel, IBAN, CB, **NIR** |
| **Point de blocage** | Avant la boucle ReAct (bloque cote agent) | Dans l'endpoint API (retourne HTTP 400) |
| **Tests** | 27 | 17 |
| **Total** | **44 tests, tous passent** | |

### Limites connues

- Filtrage par **regex** : bloque les attaques connues et variantes evidentes, mais un attaquant creatif pourrait contourner (encodage Unicode, reformulations subtiles, langues rares)
- En production, on ajouterait un **modele de classification dedie** ou un service type WAF pour LLM
- Les patterns doivent etre **maintenus** et enrichis au fil des nouvelles techniques d'attaque
