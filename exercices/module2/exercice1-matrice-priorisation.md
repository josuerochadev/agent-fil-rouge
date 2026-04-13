## Exercice 1 — Construire une matrice de priorisation  
## Durée : 25 minutes  
## ____________________________________________________________  
##   
## Étape 1 — Brainstorming assisté par LLM  
## Utilisez Claude ou ChatGPT avec ce prompt adapté à VOTRE contexte :  
Tu es un consultant en transformation digitale. Mon entreprise est dans le secteur [votre secteur]. Mon service est [votre service : RH, support, commercial, etc.].  Liste 8 processus ou tâches répétitives qui pourraient être partiellement ou totalement automatisés par un agent IA.  Pour chaque processus, donne : - Description en 1 phrase - Volume estimé (nb de fois par jour/semaine) - Temps moyen par occurrence - Complexité (simple/moyenne/complexe)  
## Notez les 8 processus proposés. Ajoutez-en 2 de votre propre initiative (total : 10).  
##   
## Étape 2 — Scorer chaque processus  
## Attribuez une note de 1 à 5 sur deux axes :  
## Axe Valeur Métier (1=faible, 5=critique) : gain de temps, satisfaction client, réduction erreurs, volume  
## Axe Faisabilité Technique (1=très difficile, 5=facile) : données disponibles, règles claires, intégration SI simple, risque RGPD faible  

| # | Processus | Valeur (1-5) | Faisabilité (1-5) | Score (V × F) |
| -- | ------------------------------------------ | ------------ | ----------------- | ------------- |
| 1 | Tri et qualification des CV ★ fil rouge | 5 | 5 | 25 |
| 2 | Réponses automatiques aux candidats | 3 | 5 | 15 |
| 3 | Pré-qualification téléphonique | 4 | 3 | 12 |
| 4 | Mise à jour du dossier de compétences | 4 | 4 | 12 |
| 5 | Onboarding administratif | 3 | 5 | 15 |
| 6 | Sourcing et approche LinkedIn / job boards | 4 | 3 | 12 |
| 7 | Suivi des consultants en intercontrat | 4 | 4 | 16 |
| 8 | Matching consultant ↔ appel d'offre | 5 | 2 | 10 |
| 9 | Génération de fiches de poste | 3 | 5 | 15 |
| 10 | Analyse des entretiens de fin de mission | 3 | 2 | 6 |
  
**Raisonnement des scores clés :**  
Le processus #1 (tri et qualification des CV) obtient 5×5=25 — c'est le seul à scorer le maximum sur les deux axes. Valeur maximale parce que c'est la tâche la plus chronophage du recrutement en ESN (volume élevé, répétitif, critères techniques bien définis). Faisabilité maximale parce que les données existent déjà (CV en entrée), les règles sont codifiables (stack requis, années d'expérience, localisation), et l'intégration SI est simple (output = liste scorée dans l'ATS).  
Le #10 (analyse des entretiens de fin de mission) prend le score le plus bas : données peu structurées, recueil souvent verbal ou informel, RGPD sensible (données collaborateurs), et valeur métier limitée — c'est un "nice to have" sans impact direct sur le pipeline.  
  
  
## Étape 3 — Sélectionner votre projet fil rouge  
## Placez vos 10 processus dans la matrice 2×2 :  
## • Quadrant haut-droite (valeur élevée + faisabilité élevée) = PRIORITÉ 1  
Tri et qualification des CV ★ fil rouge  
Mise à jour du dossier de compétences  
Suivi des consultants en intercontrat  
## • Quadrant haut-gauche (valeur élevée + faisabilité faible) = À EXPLORER  
Pré-qualification téléphonique  
Sourcing et approche LinkedIn / job boards  
Matching consultant ↔ appel d'offre  
## • Quadrant bas-droite (valeur faible + faisabilité élevée) = QUICK WIN  
Réponses automatiques aux candidats  
Onboarding administratif  
Génération de fiches de poste  
## • Quadrant bas-gauche = ABANDONNER  
Analyse des entretiens de fin de mission  
  
## Sélectionnez le processus #1 qui sera votre projet fil rouge pour le reste de la formation.  
Tri et qualification des CV ★ fil rouge  
  
Le processus retenu est le **tri et qualification automatique des CV** (score 25/25). C'est le cas d'usage idéal pour un agent IA en contexte ESN : le volume est élevé (30 à 80 CV/jour en période active), les critères de qualification sont explicites et techniques (stack, séniorité, mobilité), et aucune donnée sensible n'est produite en sortie — seulement un score et un shortlist. La faisabilité est maximale car les outils nécessaires (parsing de CV, scoring LLM, connexion ATS) sont matures et disponibles. Ce processus permettra d'illustrer tout le cycle de conception d'un agent : définition des règles métier, design du prompt, gestion des cas limites, et mesure de l'impact.  
  
## Livrable  
## Tableau des 10 processus avec scores  
## Sélection argumentée du processus fil rouge (3-4 phrases)  
##   
