# Exercice 5 — Quelle stratégie pour mon cas ?  
____________________________________________________________  
Pour chaque scénario, répondez à 4 questions :  
1. Quelle stratégie ? Prompting / RAG / Fine-tuning  
2. Pourquoi ? (2-3 arguments)  
3. Quel risque principal ?  
4. Première action concrète à lancer  
## Scénario A — Cabinet d'avocats  
Un cabinet d'avocats parisien (15 avocats) veut que ses collaborateurs puissent poser des questions en langage naturel sur la jurisprudence française. Ils disposent de 5 000 documents juridiques internes. Le budget est modéré. La précision est critique.  

| Question | Votre réponse |
| ----------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Stratégie choisie | Fine-tuning |
| Justification | La jurisprudence française mobilise un vocabulaire et un raisonnement juridique très spécifiques que le modèle de base ne maîtrise pas nativement. Le fine-tuning sur les 5 000 documents formate le modèle pour raisonner comme un juriste, pas seulement retrouver des passages. La précision critique exige une interprétation correcte, pas juste une récupération d'extraits. |
| Risque principal | Dataset d'entraînement biaisé ou incomplet — si les 5 000 documents ne couvrent pas certains domaines du droit, le modèle sera peu fiable sur ces angles précis sans le signaler. |
| Première action | Classifier les 5 000 documents par domaine juridique et évaluer leur homogénéité avant de constituer le dataset d'entraînement. |
  
    
## Scénario B — E-commerce  
Un e-commerce de mode (2 000 produits) veut générer automatiquement les descriptions de ses fiches produits dans un ton de marque très spécifique : décontracté, inclusif, avec de l'humour. Ils ont un guide de style de 10 pages et 500 exemples de descriptions déjà rédigées.  

| Question | Votre réponse |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Stratégie choisie | RAG |
| Justification | Les 500 exemples existants s'injectent directement en contexte : à chaque génération, on récupère les descriptions les plus proches stylistiquement du produit à traiter. Le modèle imite le ton sans entraînement coûteux. À 2 000 produits, le RAG est plus flexible et moins cher que le fine-tuning. |
| Risque principal | Dérive de ton si les exemples récupérés ne sont pas représentatifs du produit à décrire — une robe de soirée ne doit pas hériter du ton d'une description de streetwear. |
| Première action | Tester la recherche sémantique sur 20 produits variés pour vérifier que les exemples récupérés sont stylistiquement et catégoriellement pertinents. |
  
    
## Scénario C — Support IT  
Le service IT d'une grande entreprise (8 000 employés) reçoit 200 tickets par jour. 70 % sont des demandes simples (reset mot de passe, accès VPN, installation logiciel). Ils ont un wiki interne de 300 pages de procédures.  

| Question | Votre réponse |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Stratégie choisie | RAG |
| Justification | Le wiki de 300 pages contient toutes les réponses aux 70 % de tickets simples — inutile de fine-tuner quand la connaissance est déjà documentée et structurée ; le RAG permet de mettre à jour les procédures sans réentraîner le modèle ; le prompting seul ne suffit pas car 300 pages dépassent la fenêtre de contexte. |
| Risque principal | Procédures wiki obsolètes ou contradictoires — si une procédure de reset VPN a changé sans mise à jour du wiki, le modèle donnera des instructions incorrectes à 8 000 employés. La qualité du RAG dépend directement de la qualité de la source. |
| Première action | Auditer le wiki pour identifier les pages non mises à jour depuis plus de 6 mois, les corriger en priorité, puis indexer uniquement les pages validées avant de connecter le LLM. |
  
    
## Livrable  
Fiche avec les 3 scénarios analysés et justifiés.  
