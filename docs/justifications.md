# Justifications — Points absents du PFE

## 1. Pourquoi pas d'apprentissage auto-supervisé ?

L'apprentissage auto-supervisé (SSL) nécessite des **données non étiquetées en grande quantité**
pour fonctionner efficacement (typiquement 10x plus que le supervisé).

Dans notre cas :
- Les 3 datasets utilisés (HAPT, WISDM, PAMAP2) sont **entièrement étiquetés**
- Le volume total (56 861 fenêtres) est suffisant pour le supervisé mais marginal pour le SSL
- Les méthodes SSL pour séries temporelles IMU (TNC, TS-TCC, SimCLR-TS) nécessitent
  des augmentations spécifiques (jitter, scaling, permutation) qui peuvent altérer
  les patterns d'activité et introduire des artefacts

**Notre choix** : l'apprentissage supervisé avec perte contrastive (Module 1)
capture déjà l'essentiel des avantages du SSL (séparation inter-classes, 
représentations robustes) sans ses contraintes de données.

**Perspective future** : intégrer un pré-entraînement SSL sur des données IMU
non étiquetées (ex. données brutes de smartwatch) avant le fine-tuning supervisé.

---

## 2. Pourquoi pas de modèles de fondation pré-entraînés ?

Les modèles de fondation pour les séries temporelles IMU sont un domaine
**très récent (2023-2024)** et présentent plusieurs limitations dans notre contexte :

| Modèle | Problème |
|--------|----------|
| TimesNet, PatchTST | Conçus pour la prévision, pas la classification |
| UniTS | Domaine général, pas spécifique aux capteurs IMU |
| IMU-Transformer | Pré-entraîné sur des données propriétaires non disponibles |

**Notre choix** : entraîner un Transformer from scratch sur nos 3 datasets
permet un contrôle total sur la représentation apprise, une adaptation
directe à la structure temporelle de l'IMU (50 Hz, 6 canaux), et 
évite les problèmes de distribution shift entre le domaine de pré-entraînement
et nos données.

**Perspective future** : évaluer MOMENT ou UniTS comme backbone alternatif
dans un cadre de transfer learning.

---

## 3. Contribution originale — Replay pondéré par incertitude

Notre contribution principale est le **Uncertainty-Weighted Replay** :

Au lieu de tirer aléatoirement des exemples du tampon de rejeu :
```
Replay classique   : P(x) = 1/|M|   (uniforme)
Notre contribution : P(x) ∝ H(f_θ(x))   (entropie de prédiction)
```

**Intuition** : quand un modèle commence à oublier une classe,
il devient d'abord *incertain* sur ces exemples avant de les classer
complètement mal. En les rejouant en priorité, on intervient
exactement au bon moment.

**Paramètre α** : contrôle l'agressivité de la pondération.
- α=0 : replay uniforme (équivalent classique)
- α=1 : proportionnel à l'entropie
- α>1 : encore plus focalisé sur les exemples incertains
