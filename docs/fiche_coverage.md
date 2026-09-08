# Couverture de la fiche PFE — audit du 8 septembre 2026

Présence de code ≠ validation de l'objectif. Voir `audit_pfe_2026-09-08.md`.

| Objectif de la fiche | État constaté |
|---|---|
| HAR inertielle, Transformer | Implémentation présente ; données corrigées à régénérer |
| Nouveaux utilisateurs et nouvelles classes | Scénarios présents ; partitions et pré-entraînement à consolider |
| Mémoire externe et maintien des connaissances | Rejeu et prototypes présents ; performances à réévaluer |
| Adaptation contextuelle | CORAL présent ; comparaison à budget de labels égal à documenter |
| Auto-supervision | SSL contrastif présent ; exclusion du test avant SSL à imposer |
| Modèle de fondation | Petit pipeline pré-entraînement/adaptation, pas un modèle de fondation démontré |
| Calibration en ligne | Module présent ; feedback étiqueté nécessaire, validation dédiée manquante |
| Anticipation d'activités | Tête LSTM présente ; sessions, chevauchement et horizon futur non sécurisés |
| Prévention de perte d'équilibre | Non validée : transitions posturales ≠ perte d'équilibre |
| Préparation de repas | Non couverte par les classes conservées dans le mapping actuel |
| Renforcement | Démonstrateur de bandit de seuil, pas une validation préventive |

La fiche propose plusieurs familles de méthodes : elle n'oblige pas à implémenter
toutes les variantes ni à appeler un petit encodeur un modèle de fondation.
