# Upsilon — stratégie et état du projet

> Document de référence du projet. À lire en premier dans toute nouvelle conversation
> (Claude, Claude Code, Codex). À mettre à jour à chaque décision importante.
> Dernière mise à jour : 2026-11-11.

## 1. Objectif et cadre

- **Projet** : faire gagner de l'argent à une IA avec Baptiste comme simple interface
  (identité, comptes, paiements). La valeur doit venir du travail de l'IA, pas du temps de Baptiste.
- **Piste retenue** : un bot de prévision, **Upsilon**, dans les tournois de bots Metaculus (FutureEval).
- **Contraintes** : légal en France ; Baptiste est étudiant en L1 (les études passent en premier,
  temps disponible surtout le week-end) ; pas d'interaction client.
- **Budget** : 500 € au total pour l'expérience, dont **~200 € alloués au bot**
  (financés par les primes de bienvenue).
- **Point d'étape** : début décembre 2026 (premiers résultats MiniBench), puis fin janvier 2027
  (décision de continuer ou non).

## 2. Comment on gagne de l'argent (et quand ça n'en vaut pas la peine)

- **Notation** : score log ; *peer score* = (ton score − moyenne des autres bots) × 100.
  Un bot dans la moyenne marque ≈ 0. Score de tournoi = **somme** des peer scores,
  une question non prévue compte 0.
- **Prix ∝ score² (scores positifs seulement)** → l'argent se concentre en haut du classement.
- **Estimation (modèle grossier calé sur l'été 2026, 277 bots, cagnotte 50 k$)** :
  à 2,60 $/question sur une saison (~670 $ de coûts), on n'est rentable que vers le **top 25**
  (top ~10 %). À notre coût actuel (~0,50 $/q), le seuil de rentabilité est plus bas,
  mais il faut toujours battre nettement la moyenne.
- **Saison d'automne 2026** : on ne la joue pas (arrivée tardive : 1/3 des questions → gain
  max divisé par ~9).
- **MiniBench** (manches de 2 semaines, ~60 questions, cagnotte 1 000 $, repart de zéro) :
  pas rentable en soi ; sert à **tester le bot en conditions réelles** et à **obtenir les crédits
  gratuits** de la saison suivante (attribués selon les résultats MiniBench).
- **Vraie cible** : la saison qui commence en **janvier 2027**.

## 3. État technique actuel

- **Base** : fork de `No-Stream/metaculus-bot` (licence MIT ; 15e/277 à l'été 2026, 11e au printemps).
  Dépôt : `Kestrel-005/nostreambot-metaculus-bot`. Compte bot Metaculus : **Upsilon**.
- **Secrets GitHub configurés** : `METACULUS_TOKEN`, `OPENROUTER_API_KEY` (limite 25 $), `FRED_API_KEY`.
- **Modifications faites** (PR #1 à #3) :
  - seul **MiniBench** tourne automatiquement (toutes les 30 min) ; tournoi, Metaculus Cup et
    Mantic sont manuels ;
  - désactivé faute de clé : recherche Gemini, gap-fill v1/v2 (Gemini/Exa), lecteur payant url_context ;
  - AskNews absent → le bot bascule tout seul en mode sans AskNews ;
  - prévisionnistes : GPT-6.1-sol en effort `high` (au lieu de `xhigh`), Opus 5.5 en `xhigh`,
    Gemini 4 Argon ; agrégation par médiane.
- **Coût mesuré** : **~0,51 $/question** (test du 11/11 : 2,05 $ pour 4 questions).
  Une manche MiniBench ≈ 30 $.
- **Problèmes connus** : installation Playwright parfois en échec (passager, sans gravité) ;
  lecture des sources de résolution en échec partiel (3/5 au test) ; 3 tests de capture de
  documents échouent en local mais passent sur la CI GitHub.

## 4. Calendrier

| Quand | Quoi |
|---|---|
| Maintenant → début déc. | MiniBench tourne seul. Surveiller le solde OpenRouter (recharger ~10 $ quand il approche 0). |
| Début décembre | Bilan MiniBench : classement, coût réel, erreurs. Décider d'engager ou non le reste du budget. |
| **1er janvier 2027** | Demander l'accès AskNews (Discord AskNews, salon #api-support) + les crédits gratuits (formulaire Metaculus, rouvre avant la saison). |
| Janvier 2027 | Réactiver les sources coupées, backtests, lancer la saison (réactiver la planification du tournoi). |
| Fin janvier 2027 | Point global : continuer ou arrêter. |

## 5. Ce qui marche chez les meilleurs bots (sources publiques)

- Gagnant du printemps 2026 (GreeneiBot2, 18,9 pts/question) : plusieurs recherches en parallèle,
  distributions numériques construites proprement, vérification avant publication,
  plusieurs exécutions combinées.
- Selon l'auteur de nostreambot, par ordre d'impact : (1) plusieurs modèles différents + médiane,
  (2) bon traitement des questions numériques, (3) fiabilité (rater 10 questions ≈ −150 pts ;
  une erreur d'unité ≈ −80 pts), (4) plus de sources.
- Peu utile : retravailler les prompts, « stacking », laisser l'IA analyser ses propres erreurs.
- Les bots du top 15 dépensaient ~1,40 $/question contre ~0,50 $ pour les autres (corrélation).
- Les versions « réflexion élevée » battent leurs jumelles standard (8 paires sur 8 chez les bots Metaculus).
- Combiner les 2 meilleurs bots bat le meilleur seul.
- Le fournisseur de recherche choisi compte peu ; la **diversité** des sources compte.

## 6. Méthode d'optimisation et limites

- Une modification à la fois → **backtest** sur des questions résolues → on garde seulement si
  l'écart est net → suivi en réel sur MiniBench.
- **Mesure = contrainte principale** : écart de score très bruité (±20 pts/question) ;
  détecter +2 pts demande ~400 questions ≈ 200 $ par expérience.
- **Fuite d'information** :
  - recherche web qui renvoie des articles postérieurs → besoin de données datées (AskNews) ;
  - **les modèles connaissent les réponses résolues avant leur date de coupure** → seules les
    questions résolues après la coupure sont utilisables en backtest.
- **Surajustement** : garder des questions de côté jamais utilisées pendant les réglages.
- Rendements décroissants : on part d'un bot déjà très optimisé.

## 7. Idées à tester (par priorité)

1. **Fiabilité** : vérifier dans les exécutions MiniBench qu'aucune question n'est ratée ou mal publiée. *(gratuit)*
2. **Réactiver les sources coupées** dès que possible (AskNews en janvier ; clé Gemini à évaluer).
3. **Vérificateur des critères de résolution** : relire dates, unités, conditions avant publication
   (cause fréquente d'erreurs catastrophiques).
4. **Questions numériques** : largeur des distributions, queues.
5. **Modules par domaine** : finance via les prix d'options (distribution implicite),
   sport (Elo), élections (moyennes de sondages), météo (prévisions officielles).
6. **Base de cas similaires** : taux de base chiffrés à partir de questions passées comparables.
7. **Poids du prix des marchés de prédiction** selon la liquidité et la proximité avec la question.
8. **Combiner deux bots différents** (coût doublé).
9. **Choix des modèles / niveau de réflexion** : à mesurer en backtest (ex. GPT high vs xhigh).

Pistes écartées : construire notre propre AskNews avant janvier (GDELT + Wayback Machine) —
faisable mais pas rentable vu le délai ; s'inspirer des bots Polymarket (leurs gains viennent
surtout de la mécanique de marché, absente de Metaculus).

## 8. Hors bot (gains ponctuels)

- Primes de bienvenue bancaires et offres de bienvenue des bookmakers agréés ANJ :
  bonus ponctuels, à faible effort seulement (Revolut reste la banque principale, pas de
  mobilité bancaire).

## 9. Journal des décisions

- 2026-11-10 : compte bot Metaculus Upsilon créé ; demandes de crédits fermées pour l'automne.
- 2026-11-11 : budget du bot porté à ~200 € ; base = fork de nostreambot plutôt que le bot modèle
  de Metaculus ; saison d'automne non jouée ; MiniBench seul en automatique ; AskNews indisponible
  jusqu'au 1er janvier (confirmé sur leur Discord).

## Sources principales

- Calcul des prix : https://www.metaculus.com/questions/21883/introducing-simpler-fairer-tournament-scores/
- Annonce automne 2026 : https://forum.effectivealtruism.org/posts/hrJrsgMCwsdBuAqgQ/announcing-fall-2026-futureeval-bot-tournament
- Synthèse de 11 analyses : https://forum.effectivealtruism.org/posts/Spyz3wESZu2eeqhDj/ai-forecasting-in-2026-what-11-analyses-say
- Résultats printemps 2026 : https://www.lesswrong.com/posts/wZBbDqzfBjYG58CxK/futureeval-spring-results-pros-beat-bots-but-the-gap-is
- Article de l'auteur de nostreambot : https://nostream.substack.com/p/on-writing-a-bitter-pilled-metaculus
