# Exercice 3 — Spécifier les outils de votre agent  
**Durée : 25 minutes**  
____________________________________________________________  
## Étape 1 — Identifier les outils nécessaires  
Reprenez votre projet fil rouge. Listez toutes les actions que l'agent doit pouvoir réaliser :  

| # | Action | Système concerné | Type (lecture/écriture/les deux) |
| - | ------------------------------------------ | ------------------------------ | -------------------------------- |
| 1 | Récupérer les articles depuis les flux RSS | Flux RSS (internet) | Lecture |
| 2 | Filtrer/scorer les articles par pertinence | Dataset local (JSON/CSV) + LLM | Les deux |
| 3 | Grouper les articles par catégories | LLM (Claude API) | Les deux |
| 4 | Résumer un article ou un lot d'articles | LLM (Claude API) | Lecture |
| 5 | Stocker les articles résumés localment  | Dataset local | Écriture |
| 6 | Envoyer le compte-rendu par mail | SMTP / service mail | Écriture |
| 7 | Gérer la liste des sources RSS | Dataset local (config) | Les deux |
| 8 | Marquer les articles comme lus/archivés | Dataset local | Écriture |
  
    
## Étape 2 — Spécifier 3 outils en détail  
Pour les 3 outils les plus importants, remplissez le template :  
NOM : fetch_rss_feed  DESCRIPTION (pour le LLM — 1-2 phrases) : "Récupère et parse les entrées d'un flux RSS depuis une URL donnée, en filtrant optionnellement les articles déjà vus. Retourne une liste d'articles normalisés prêts à être traités."  ENTRÉES : url (string, obligatoire) : URL du flux RSS à interroger  
since (string ISO 8601, optionnel) : date plancher — ne retourne que les articles publiés après cette date  
max_items (int, optionnel, défaut : 20) : nombre maximum d'articles à retourner  
  SORTIE (succès) — format JSON : {  
  "source_url": "https://example.com/rss",  
  "fetched_at": "2026-04-09T10:00:00Z",  
  "items": [  
    {  
      "id": "hash_unique",  
      "title": "Titre de l'article",  
      "url": "https://...",  
      "published_at": "2026-04-08T14:30:00Z",  
      "summary": "Résumé extrait du flux",  
      "tags": ["AI", "Python"]  
    }  
  ]  
}  SORTIE (erreur) — format JSON : {  
  "erreur": "feed_unreachable",  
  "url": "https://example.com/rss",  
  "detail": "HTTP 404"  
}  GESTION D'ERREUR : URL invalide → retourner invalid_url, ne pas planter l'agent  
Feed mal formé / pas du RSS valide → parse_error, loguer et continuer avec les autres sources  
Timeout (> 10s) → timeout, l'agent réessaie une fois puis passe à la source suivante  
  
**Outil 2**  
**NOM :** score_and_filter_articles  
**DESCRIPTION :** Évalue la pertinence de chaque article par rapport aux thématiques de veille configurées, lui attribue un score, et retourne uniquement ceux qui dépassent le seuil défini.  
**ENTRÉES :**  
* articles (array d'objets, obligatoire) : liste d'articles au format de sortie de fetch_rss_feed  
* topics (array de strings, obligatoire) : thématiques cibles (ex. ["COBOL", "mainframe", "C#", "IA générative"])  
* threshold (float 0–1, optionnel, défaut : 0.5) : score minimum pour qu'un article soit retenu  
**SORTIE (succès) :**  
{  
  "filtered": [  
    {  
      "id": "hash_unique",  
      "title": "...",  
      "url": "...",  
      "score": 0.82,  
      "matched_topics": ["COBOL", "mainframe"]  
    }  
  ],  
  "total_input": 20,  
  "total_retained": 5  
}  
  
**SORTIE (erreur) :**  
{  
  "erreur": "empty_input",  
  "detail": "La liste d'articles fournie est vide"  
}  
  
**GESTION D'ERREUR :**  
* Liste vide → empty_input, retourner un résultat vide sans appel LLM  
* topics vide → missing_topics, bloquer et demander une configuration valide  
* Timeout scoring LLM (> 15s) → scoring_timeout, retourner les articles avec score null pour traitement manuel  
  
**Outil 3**  
**NOM :** send_digest_email  
**DESCRIPTION :** Génère et envoie un mail de compte-rendu formaté à partir d'une liste d'articles scorés et de leurs résumés. Supporte HTML et texte brut.  
**ENTRÉES :**  
* recipient (string, obligatoire) : adresse mail du destinataire  
* subject (string, optionnel, défaut : "Veille techno — [date]") : objet du mail  
* articles (array d'objets, obligatoire) : articles à inclure, avec titre, url, score et résumé  
* format (string "html"|"text", optionnel, défaut : "html") : format du corps du mail  
**SORTIE (succès) :**  
{  
  "sent": true,  
  "recipient": "josue@example.com",  
  "sent_at": "2026-04-09T10:05:00Z",  
  "articles_count": 5  
}  
**SORTIE (erreur) :**  
{  
  "erreur": "smtp_auth_failure",  
  "detail": "Identifiants SMTP invalides ou expirés"  
}  
  
**GESTION D'ERREUR :**  
* Adresse invalide → invalid_recipient, bloquer l'envoi et alerter dans les logs  
* Échec SMTP → smtp_failure, réessayer 2 fois avec backoff de 30s, puis smtp_max_retries  
* Liste d'articles vide → empty_digest, ne pas envoyer et loguer un warning  
* Timeout (> 20s) → send_timeout, marquer l'envoi comme pending pour retry ultérieur  
  
  
## Étape 3 — Validation croisée avec le LLM  
Copiez vos 3 spécifications dans Claude ou ChatGPT avec ce prompt :  
Voici les spécifications de 3 outils pour un agent IA. Analyse chaque spécification et identifie : 1. Les cas d'erreur manquants 2. Les ambiguïtés dans la description 3. Les paramètres manquants 4. Un scénario où le LLM appellerait le mauvais outil  
Notez les améliorations suggérées et corrigez vos spécifications.  
  
Voici les spécifications de 3 outils pour un agent IA de veille technologique.  
Analyse chaque spécification et identifie :  
1. Les cas d'erreur manquants  
2. Les ambiguïtés dans la description  
3. Les paramètres manquants  
4. Un scénario où le LLM appellerait le mauvais outil  
  
--- OUTIL 1 : fetch_rss_feed ---  
[colle la spec complète]  
  
--- OUTIL 2 : score_and_filter_articles ---  
[colle la spec complète]  
  
--- OUTIL 3 : send_digest_email ---  
[colle la spec complète]  
  
  
## Livrable  
* Liste complète des outils nécessaires (5 minimum)  
* 3 outils spécifiés en détail avec le template  
* Feedback du LLM et corrections apportées  
