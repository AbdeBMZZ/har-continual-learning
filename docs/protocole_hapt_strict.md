# Protocole HAPT temporel — deuxième lot de corrections

Ce lot complète l'audit du 8 septembre 2026. Il prépare une expérience reproductible de reconnaissance supervisée puis de classification d'une fenêtre future. Il ne valide pas encore l'ensemble du sujet PFE ni la prévention des chutes.

## Garanties implémentées

- Entrée : fichiers HAPT RawData originaux, annotations conservées dans leur chronologie. Les anciennes matrices fusionnées ne permettent pas de retrouver les frontières de sessions et ne conviennent pas à ce protocole.
- Enregistrements et positions des fenêtres conservés ; aucun raccord à travers les fenêtres manquantes ou les intervalles non annotés.
- Filtre causal à 50 Hz, retrait de la gravité sur l'accélération uniquement, abandon des 500 premiers échantillons de chaque session. Pas de rééchantillonnage global ni de filtrage avant-arrière. L'unité de l'accélération doit être vérifiée dans la documentation des données téléchargées.
- Répartition fixe par sujets, environ 60 % apprentissage, 20 % validation, 20 % test. Normalisation ajustée sur les sujets d'apprentissage uniquement, puis enregistrée dans le checkpoint.
- Identifiant du protocole et empreintes des tableaux ; refus des anciens checkpoints sans cette provenance et des sorties déjà existantes.
- Séquences de préfixes observés. La fenêtre cible commence après la fin de la dernière fenêtre complète du contexte plus l'horizon demandé, arrondi à la grille des fenêtres. Les indices cibles restent identiques entre ratios d'observation : les petits ratios ont donc un horizon effectif plus long, exporté dans les résultats.
- Tous les exemples temporels valides sont conservés : aucune sélection à partir de transitions futures connues.
- Entraînement sur apprentissage, choix de l'époque sur validation, commande séparée pour le test. Le modèle de reconnaissance est entraîné de zéro sur ce même partage ; le backbone est ensuite figé pour l'anticipation.
- Macro-F1 calculé sur les 12 classes HAPT fixes, classes absentes incluses avec score nul. Référence constante déterminée sur les cibles d'apprentissage, jamais sur celles du test.

## Utilisation

Depuis la racine du dépôt, dans un environnement avec les dépendances du projet et PyTorch. Remplacer `UNITE` par `g` ou `ms2` après vérification, et le chemin RawData par le chemin réel.

```bash
python scripts/prepare_hapt_protocol.py --raw /chemin/HAPT/RawData --acc-unit UNITE --out data/hapt_strict_v1
python scripts/run_hapt_protocol.py --protocol data/hapt_strict_v1 --stage recognition --epochs 20 --out checkpoints/hapt_recognition.pt
python scripts/run_hapt_protocol.py --protocol data/hapt_strict_v1 --stage anticipation --checkpoint checkpoints/hapt_recognition.pt --ratio 0.5 --epochs 20 --out checkpoints/hapt_anticipation_p50.pt
python scripts/run_hapt_protocol.py --protocol data/hapt_strict_v1 --stage test --checkpoint checkpoints/hapt_anticipation_p50.pt --out results/hapt_anticipation_p50_test.json
```

Choisir les hyperparamètres sur validation avant de consulter le test. Pour comparer des ratios, conserver les mêmes paramètres de séquence/horizon et produire un checkpoint par ratio. La commande test reprend les paramètres enregistrés. Le checkpoint de reconnaissance utilise la tête de classification supervisée ; cette expérience ne mesure pas la mémoire de prototypes ni l'apprentissage continu.

## Vérification et limites

14 tests réussis, 1 test PyTorch ignoré car PyTorch est absent de l'environnement d'audit. Compilation syntaxique et aide de la nouvelle commande vérifiées. Les tests couvrent causalité, préfixes, horizon, interruptions, sessions, partition des sujets, empreintes et rejet des checkpoints hérités. Aucun entraînement réel ni score scientifique nouveau produit. Le nouveau lanceur d'entraînement doit encore être exécuté et vérifié dans un environnement PyTorch.

Le protocole porte exclusivement sur HAPT, avec les classes originales 1 à 12. Il mesure le label majoritaire d'une fenêtre future, pas le délai jusqu'au prochain changement d'activité. HAPT ne fournit pas ici une validation de pertes d'équilibre ou de chutes. Le retrait de 10 secondes au démarrage et l'exclusion d'intervalles non annotés doivent être décrits et leur impact quantifié dans le rapport.

Restent à traiter : protocole séquentiel d'apprentissage continu avec classes réellement nouvelles, comparaisons sans adaptation/finetuning/replay, références jointes pour l'intransigeance, répétitions sur plusieurs graines, correction du réservoir et de la couverture des prototypes, provenance d'un éventuel pré-entraînement auto-supervisé. Les anciennes commandes ne sont pas automatiquement converties au nouveau protocole ; leurs anciens résultats ne doivent pas être mélangés aux nouveaux.

Le LaTeX n'est pas modifié dans ce lot. Sa structure et sa typographie seront alignées sur le mémoire de Mazouz et Oukil après stabilisation du protocole et obtention de résultats réels.
