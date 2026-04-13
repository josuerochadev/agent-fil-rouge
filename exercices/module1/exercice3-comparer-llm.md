# Exercice 3 — Comparer les LLM sur des questions opinionnées  
Matériel : accès à GPT-4o, Claude et Mistral (ou 2 sur 3 minimum)  
____________________________________________________________  
## Étape 1 — Poser les 3 questions  
Posez exactement le même prompt à chaque modèle (copiez-collez pour garantir l'identité) :  
**Question 1 — Éthique :**  
"Le télétravail devrait-il être un droit légal pour tous les employés de bureau en France ? Argumente avec des points pour et contre, puis donne ta recommandation."  
**Question 2 — Décision métier :**  
"Je dirige un service client qui reçoit 500 réclamations par jour. 60 % sont répétitives (mot de passe, suivi commande, FAQ). Dois-je automatiser les réponses avec un LLM ? Quels sont les risques ? Donne-moi un plan d'action concret."  
**Question 3 — Créative :**  
"Écris 3 slogans pour une startup d'IA éthique spécialisée en santé mentale. Le ton doit être humain et rassurant, pas technologique."  
## Étape 2 — Remplir les grilles d'analyse  
Pour chaque question :  

| Critère | GPT-4o | Claude | Mistral |
| -------------------------------- | ----------------------------------------------------- | ---------------------------------------------------------- | ----------------------------------------------------- |
| Prend position ou reste neutre ? | Prend position sur Q1 et Q2, neutre sur Q3 par nature | Prend position sur Q1 et Q2, neutre sur Q3 par nature | Prend position sur Q1, neutre sur Q2 et Q3 par nature |
| Niveau de détail opérationnel | Modéré pour les trois questions | Élevé sur Q2 (phases, indicateurs, risques), modéré sur Q1 | Modéré pour les trois questions |
| Créativité / originalité | Très sobre et pas trop original  | Slogans volontairement sobres, évitent le jargon tech | Très sobre et pas trop original  |
| Ton et style | Direct, structuré | Direct, structuré | Direct, structuré |
| Note qualité /5 | 4 | 4.5 | 3 |
  
    
## Étape 3 — Synthèse  
Répondez à ces questions :  
1. Quel modèle est le plus prudent ? (refuse de prendre position, ajoute des nuances)  
2. Quel modèle est le plus actionnable ? (donne des plans concrets, des chiffres)  
3. Quel modèle est le plus créatif ? (surprend, sort du générique)  
4. Observez-vous des biais culturels ? (référence au contexte français/européen vs américain)  
  
Les trois modèles partagent un style direct et structuré, mais se différencient nettement sur deux critères clés : le détail opérationnel et la prise de position.  
Sur les questions éthiques (Q1), GPT-4o et Claude prennent position tandis que Mistral préfère équilibrer sans trancher — ce qui le rend plus prudent mais moins utile quand une décision est attendue. Sur les questions métier (Q2), Claude se distingue clairement avec un niveau de détail supérieur : phases chiffrées, indicateurs de succès, risques RGPD — là où GPT-4o et Mistral restent dans des recommandations générales. Sur la créativité (Q3), aucun modèle ne se démarque vraiment : les trois produisent des slogans corrects mais prévisibles, sans vraie prise de risque stylistique.  
Mistral obtient la note la plus basse (3/5), reflétant une tendance à la neutralité qui limite son utilité dans des contextes où une recommandation concrète est attendue. GPT-4o (4/5) est solide et fiable mais générique. Claude (4,5/5) se distingue principalement sur les cas d'usage métier complexes.  
  
## Livrable  
Recommandation argumentée (½ page) :  
"Pour un usage [éthique/métier/créatif], je recommande [modèle] car [justification basée sur l'exercice]."  
  
"Pour un usage **métier**, je recommande **Claude** car il est le seul des trois à produire un plan d'action structuré avec des phases, des indicateurs et des risques identifiés — ce qui est directement exploitable en contexte professionnel sans retraitement."  
"Pour un usage **éthique**, je recommande **GPT-4o** car il prend position clairement tout en contextualisant au cadre français, avec un niveau de détail comparable à Claude mais une formulation plus accessible."  
"Pour un usage **créatif**, aucun modèle ne se distingue significativement — les trois restent dans un registre sobre et attendu ; dans ce cas, le choix dépendra davantage des préférences de style de l'utilisateur que des capacités intrinsèques du modèle."  
