# Exercice 2 — Choisir le bon pattern d'agent  
____________________________________________________________  
## 4 cas à analyser  
Pour chaque cas, choisissez le pattern (réactif / ReAct / planificateur) et justifiez :  
**Cas A — Chatbot FAQ interne**  
Les employés posent des questions sur les congés, les notes de frais et les procédures RH. Les réponses sont dans un wiki de 50 pages. L'agent doit répondre en < 5 secondes.  

| Question | Votre réponse |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pattern choisi | Réactif |
| Justification | Les questions sont prévisibles et récurrentes. On doit juste chercher dans le wiki et reformuler. Pas de décision conditionnelle, pas d'étapes enchaînées. La contrainte < 5 secondes exclut toute boucle de raisonnement. |
| Pourquoi PAS les 2 autres | ReAct implique des cycles observe → think → act qui ajoutent de la latence. Inutile ici : une seule recherche suffit. Le planificateur décompose un objectif complexe en sous-tâches — surqualifié pour une FAQ statique. Lent et coûteux en tokens. |
| Outils nécessaires | Recherche sémantique sur le wiki (RAG), génération de réponse |
  
    
**Cas B — Qualification de leads**  
L'agent reçoit un formulaire de contact. Il doit : 1) chercher l'entreprise dans le CRM, 2) vérifier le CA sur Societe.com, 3) consulter l'historique, 4) attribuer un score, 5) créer une opportunité si score > 70.  

| Question | Votre réponse |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pattern choisi | ReAct |
| Justification | La séquence est semi-structurée : les étapes 1–3 sont fixes, mais l'étape 4 (score) conditionne l'étape 5. L'agent doit observer le résultat de chaque outil avant de décider la suite. C'est exactement le profil ReAct : boucle observe → raisonne → agit, avec branchement conditionnel. |
| Pourquoi PAS les 2 autres | Le réactif ne gère pas les dépendances entre étapes ni les conditions (score > 70). Il ne sait pas "si... alors...". Le plan est déjà connu à l'avance (5 étapes fixes). Le planificateur est utile quand l'objectif est ouvert et que les sous-tâches sont à découvrir dynamiquement — ce n'est pas le cas ici. |
| Outils nécessaires | API CRM (lecture + écriture), scraping/API Societe.com, moteur de scoring, création d'opportunité CRM |
  
    
**Cas C — Assistant rédaction d'emails**  
L'utilisateur donne une consigne ('réponds à ce client mécontent') et l'agent génère un email adapté. Pas d'accès à des systèmes externes.  

| Question | Votre réponse |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pattern choisi | Réactif |
| Justification | L'agent reçoit une instruction, génère un output en une passe, s'arrête. Zéro système externe, zéro décision conditionnelle. C'est de la génération pure : input → LLM → output. |
| Pourquoi PAS les 2 autres | ReAct sert à interagir avec des outils et à adapter le raisonnement en fonction des résultats. Ici il n'y a rien à observer ni à itérer. Il n'y a qu'une seule tâche atomique. Décomposer "écrire un email" en sous-tâches planifiées serait de l'over-engineering complet. |
  
    
**Cas D — Analyse de sinistres (assurance)**  
L'agent reçoit une déclaration (texte + photos). Il doit : classifier le type, estimer le montant, vérifier la couverture, décider (refus/acceptation/escalade), générer le courrier de réponse.  

| Question | Votre réponse |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pattern choisi | Planificateur |
| Justification | Le cas combine : entrée multimodale (texte + photos), plusieurs sous-tâches hétérogènes (classification, estimation, vérification juridique, décision, rédaction), et une décision finale à 3 branches (refus / acceptation / escalade). L'agent doit construire un plan, coordonner des modules spécialisés et adapter la suite selon les résultats intermédiaires. C'est le profil typique d'un planificateur. |
| Pourquoi PAS les 2 autres | Trop de complexité et de branchements. Le réactif ne peut pas orchestrer plusieurs agents/outils avec logique conditionnelle. ReAct est adapté à une boucle outillée simple. Ici la complexité est verticale (sous-tâches qualitativement différentes) et non pas une boucle homogène. Il faudrait un méta-agent pour coordonner — c'est le planificateur. |
| Outils nécessaires | Vision/OCR (analyse photos), modèle de classification sinistres, estimateur de coûts, base de données polices/couvertures, moteur de décision, générateur de courrier |
  
    
## Appliquer à votre projet fil rouge  

| Question                        | Votre réponse |
| ------------------------------- | ------------- |
| Votre processus                 |               |
| Nombre d'étapes                 |               |
| Systèmes externes à consulter ? |               |
| Actions à exécuter ?            |               |
| Pattern choisi                  |               |
| Justification                   |               |
  
    
## Livrable  
* Les 4 cas analysés avec justification  
* Le pattern choisi pour votre projet fil rouge avec justification (projet à definir)  
