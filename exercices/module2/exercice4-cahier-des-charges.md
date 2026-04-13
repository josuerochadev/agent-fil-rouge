# Exercice 4 — Rédiger le cahier des charges de votre agent

**Équipe** : Alexis Dubus - Josué X. Rocha - Zhenfeng DING - Stéphanie Consoli
**Durée** : 45 minutes

---

## 1. Problème métier

| Élément | Réponse |
|---|---|
| Processus cible | Veille technologique quotidienne |
| Volume actuel (nb/jour) | 2 articles / jour / personne |
| Temps moyen par occurrence | 15 minutes par article |
| Coût actuel estimé | (15×4) = 1h ; 22 jours/mois × 1200 employés × 39,8 €/h = **6 829 680 €/an** |
| Problème principal | La veille technologique est nécessaire, mais elle n'est souvent pas une priorité. Par manque de temps, d'intérêt ou de priorité. |

## 2. Objectifs

| Élément | Réponse |
|---|---|
| Objectif principal | Automatiser la collecte, le filtrage, la synthèse et la diffusion de la veille technologique quotidienne |
| Gain attendu (quantifié) | Réduction du temps de veille manuelle de 80-90% (lecture et digest). Couverture des sources surveillées sans effort |
| Périmètre inclus | RSS, filtre par thème, résumé LLM, groupement par catégorie, stockage local, envoi quotidien, archivage des articles traités |
| Périmètre exclu | Réseaux sociaux, traduction, interface utilisateur |

## 3. Utilisateurs cibles

| Profil | Rôle | Interaction avec l'agent | Fréquence |
|---|---|---|---|
| IT | Supervision | Maintenance | Quotidienne |
| Analyste | Validation des rapports | Corrections | Quotidienne |

## 4. Pattern agent

| Élément | Réponse |
|---|---|
| Pattern choisi | Planificateur |
| Justification | Il n'y a pas de réaction, c'est automatique |

## 5. Entrées et sorties

**Entrées acceptées :**

| Type d'entrée | Format | Exemple |
|---|---|---|
| Flux RSS | Texte/HTML | `https://next.ink/rss/news.xml`, `https://4sysops.com/feed/` |
| Liste des sources RSS | JSON | |

**Sorties produites :**

| Type de sortie | Format | Exemple |
|---|---|---|
| Mail | Texte + liens HTTP | Rapport de veille technologique sur Nvidia. Nvidia a développé une nouvelle technologie pour les puces graphiques, qui sera utilisée sur les LLMs. |
| Dataset local | JSON | |
| Logs | TXT / JSON | |

## 6. Architecture — Schéma des 7 couches

| Couche | Contenu pour votre agent |
|---|---|
| 1. Perception/Entrée | Flux RSS |
| 2. Orchestration | Pipeline : Scraping → Analyse → Filtrage + Groupement → Synthèse → Stockage → Mail |
| 3. Raisonnement (LLM) | Génération de résumé, groupement par catégorie, score de pertinence |
| 4. Mémoire | Dataset local : résumés d'articles traités, historique des envois, liste des articles archivés |
| 5. Outils | RSS parser, LLM API, SMTP, stockage JSON/SQLite |
| 6. Contrôle/Gouvernance | Gestion d'erreurs, logs, vérification RGPD |
| 7. Sortie | Fichier JSON, mail envoyé |

## 7. Données nécessaires

| Source | Type | Volume | Action RGPD |
|---|---|---|---|
| Flux RSS public | Texte / HTML | 2 articles / jour / personne | Aucune |
| Thèmes | JSON | — | Aucune |
| Pertinence | JSON | — | Aucune |
| Dataset résumé articles | JSON | — | Rétention 90j |
| Adresses emails destinataires | — | — | Consentement requis, durée limitée |
| Logs | TXT / JSON | — | Rétention 30j |

## 8. KPIs

| KPI | Cible | Méthode de mesure |
|---|---|---|
| Temps gagné par collaborateur | 3 premiers mois : -15% ; fin année 1 : -40% | Formulaire auto-déclaratif obligatoire + contrôle des temps passés sur URLs ciblées |
| Standardisation de la veille | Newsletter normée d'entreprise générée par l'agent | Taux d'ouverture, taux de clics, évolution dans le temps |
| Taux de mise en place de nouvelles technos | Comparatif avant/après | Comptage et qualification avec le contrôle de gestion ou DAF |

## 9. Parties prenantes

| Partie prenante | Rôle dans le projet | Quand l'impliquer |
|---|---|---|
| Métier | Qualification et mesure de l'impact | Cadrage, refinement, tests fonctionnels |
| IT | Développement du pipeline, validation sécurité | Tout au long du projet |
| Juridique/DPO | Contrôle compliance | Phase cadrage et livraison |
| Direction | Validation priorisation et budget | Fin de cadrage (CODIR) |

## 10. Risques identifiés

| Risque | Probabilité | Impact | Mitigation |
|---|---|---|---|
| Fausses informations | Élevé | Moyen | Auto-correction |
| RSS cassés (hors ligne) | Faible | Faible | Retry automatique |
| Surcharge d'information | Élevé | Faible | Maximum de X rapports/jour, choix de catégories |
| Non-conformité RGPD | Faible | Critique | Vérification automatique |
| Pertinence / catégories fausses | Moyen | Moyen | Tests d'intégration |
| Fuite des données (emails) | Faible | Élevé | Chiffrage en config locale |
| Dépendance technologique | Faible | Faible | Projet simple à migrer |
| Mauvaise priorisation des tendances | Élevé | Faible | Liste de sources fiable, administrée et révisée |

## Livrable

Cahier des charges complet (les 10 sections remplies). Ce document est le socle du projet fil rouge pour les modules suivants.
