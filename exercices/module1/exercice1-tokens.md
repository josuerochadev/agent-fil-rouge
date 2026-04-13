# Exercice 1 — Toucher les tokens

**Matériel** : navigateur web avec accès à platform.openai.com/tokenizer

---

## Étape 1 — Tokeniser et observer

Rendez-vous sur le tokenizer OpenAI. Collez chaque phrase et notez le nombre de tokens :

| Phrase | Nb tokens | Observation |
|---|---|---|
| "Bonjour, comment allez-vous aujourd'hui ?" | 11 | |
| "Hello, how are you doing today?" | 9 | |
| `def calculate_price(qty, price): return qty * price` | 12 | |
| "L'intelligence artificielle générative transforme les processus métiers." | 12 | |
| "AI transforms business." | 5 | |

**Question** : pourquoi le français génère-t-il plus de tokens que l'anglais pour un contenu équivalent ?

## Étape 2 — Calculer les coûts

Vous devez envoyer un document de 10 pages (~4 000 mots français) à un LLM pour en faire une synthèse. Le LLM produit une réponse de ~500 mots.

Calculez le coût pour chaque modèle :

| Modèle | Tokens input (estim.) | Coût input | Tokens output (estim.) | Coût output | Coût total |
|---|---|---|---|---|---|
| GPT-4o (2.50 $/1M in, 10 $/1M out) | 5 500 | 0,01375 $ | 700 | 0,007 $ | 0,021 $ |
| Claude Sonnet (3 $/1M in, 15 $/1M out) | 5 500 | 0,0165 $ | 700 | 0,0105 $ | 0,027 $ |
| GPT-4o mini (0.15 $/1M in, 0.60 $/1M out) | 5 500 | 0,000825 $ | 700 | 0,00042 $ | 0,00125 $ |

*Indice : 1 mot français ≈ 1.3 tokens*

## Étape 3 — Mesurer l'impact de la compaction

**Scénario** : vous avez un document de 50 pages (25 000 mots français ≈ 32 500 tokens).

| Stratégie | Tokens envoyés | Coût GPT-4o | Qualité attendue |
|---|---|---|---|
| Envoyer le document entier | 32 500 | 0,088 $ | Bonne mais risque de perte |
| Résumer en 2 pages avant envoi | ~2 600 | 0,014 $ | Perte d'information possible |
| Chunking : 5 morceaux de 10 pages | 5 x 6 500 | 0,143 $ | Bonne, mais 5x plus d'appels |

**Question** : dans quel cas chaque stratégie est-elle la plus adaptée ?

## Livrable

- Tableau comparatif rempli
- "Le français coûte 30 % de plus que l'anglais en tokens"
- "Pour mon cas d'usage, la stratégie optimale est le document entier car la fenêtre de contexte de GPT-4o (128k tokens) le permet, le coût reste faible (0,088 $), et la qualité de synthèse est meilleure qu'avec un résumé préalable qui ferait perdre de l'information."
