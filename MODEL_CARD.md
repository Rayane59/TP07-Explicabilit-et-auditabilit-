# Model Card - fraude (version servie par l'alias production)

## Modele
Nom et version dans le registre MLflow : `fraude`, version 1, alias `production` (registre du TP2, `tp2-mlflow/mlflow.db`).
Famille et reglages : RandomForestClassifier, 500 arbres, profondeur max 16, random_state 42 (scikit-learn 1.9.1, format skops).
Run d'origine (MLflow) : `f5629ebca0f94301bdcc00fd6f132617`.

## Objectif et usage prevu
Donner une probabilite de fraude a une transaction bancaire au moment du paiement, pour signaler les paiements suspects a un conseiller ou declencher une verification (SMS, appel).

## Usages exclus
Bloquer definitivement un compte ou refuser un client sans revue humaine : le modele se trompe sur 20 fraudes et 15 paiements normaux du jeu de test.
Etre applique a d'autres pays, devises ou types de paiement que ceux des donnees d'entrainement, ou utilise comme preuve de fraude.

## Donnees
Fichier, nombre de lignes, part de fraudes, empreinte (tag donnees_md5) : `data/transactions.csv`, 5 000 lignes, 568 fraudes (11,4 %), md5 `cf9f6f86c141e7df32c61715c493483f` (identique au ticket DVC v1 du TP3).
Origine des donnees (reelles ou synthetiques) : synthetiques, generees par `donnees.py` (graine 7) ; 4 variables : montant, anciennete, freq_24h, nuit.

## Performances (jeu de test, 1 250 transactions)
F1 : 0.8669   precision : 0.8837   rappel : 0.8507   AUC : 0.9872
Matrice de confusion : [[1101, 15], [20, 114]] -> 114 fraudes detectees sur 134.

## Methodes d'explicabilite
Globale : importance par permutation (10 melanges, perte de F1) et SHAP TreeExplainer sur 400 transactions de test (valeur de base 0.1157).
Locale : SHAP waterfall (Mme Benali, 1 800 MAD de nuit : P = 0.924 ; nuit +0.370, montant +0.367) et LIME (2 000 voisins, graine 0).
Variables les plus influentes : montant en tete (permutation 0.593, SHAP 0.116), puis anciennete, nuit et freq_24h tres proches. Le montant pousse vers la fraude au-dela d'environ 1 000 MAD ; une faible anciennete pousse vers la fraude.

## Comparaison avec un modele interpretable
Une regression logistique sur log10(montant) atteint F1 0.8593 (113 fraudes sur 134, contre 114 pour la foret) et se lit directement (cote x 4.76 par transaction en 24 h, x 0.928 par mois d'anciennete). Un arbre de profondeur 3 reste lisible mais tombe a F1 0.5737.

## Limites et risques connus
Donnees synthetiques : les performances ne garantissent rien sur des donnees reelles.
LIME est instable : anciennete et freq_24h changent de signe selon la graine (seuls montant et nuit sont stables) ; SHAP mesure une contribution au modele, pas une cause.
La variable nuit peut penaliser des clients qui paient legitimement la nuit : son effet doit etre surveille.

## Recommandations d'utilisation et de surveillance
Toujours une revue humaine avant blocage ; communiquer au client seulement les contributions stables (SHAP, confirmees par LIME sur plusieurs graines).
Surveiller chaque mois le F1, le taux de fraude et la distribution des variables (derive) ; reexpliquer le modele a chaque changement de version ou d'alias.

## Tracabilite
Run des explications (experience tp7-explicabilite) : `9e16cfc504dc4ae68f6d57db01f486e7` (tag `explications_run` de la version 1).
Auteurs : Rayane59 (a completer : nom, prenom, groupe)   Date : 2026-11-12   Prochaine revue : 2027-02-12
