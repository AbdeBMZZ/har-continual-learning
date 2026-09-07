# Justifications — Points absents / maintenant couverts

> Mise à jour : des modules **légers** ont été ajoutés pour coller à la fiche
> (`docs/fiche_coverage.md`). Ce qui suit reste la justification des *limites*.

## 1. Auto-supervisé (SSL)

**Avant** : non implémenté (données entièrement labellisées, volume marginal).

**Maintenant** : prétrain SimCLR-style IMU dans `scripts/train_ssl.py`
(augmentations bruit/scale/mask + NT-Xent). Utile comme étape optionnelle
avant fine-tuning ; ce n’est pas un SSL massif type industrie.

## 2. Modèles de fondation

Toujours **pas** de FM public (TimesNet/UniTS/MOMENT non intégrés).

**Maintenant** : pipeline *foundation-style* `scripts/train_foundation_style.py`
= SSL → fine-tune supervisé à l’échelle HAPT/WISDM. À présenter comme
*pattern* fondation, pas comme modèle de fondation.

## 3. Contribution originale — Replay pondéré par incertitude

Notre contribution principale reste le **Uncertainty-Weighted Replay** :

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

## 4. RL, chute, repas, calibration

| Point | Couverture |
|-------|------------|
| RL | Bandit ε-greedy sur seuil d’alerte (`rl_threshold_agent.py`) |
| Chute préventive | Anticipation binaire des transitions 9–12 + heuristique physique |
| Repas | Non entraînable sans corpus cuisine (`adl_scenarios.py`) |
| Calibration online | Temperature scaling + seuil adaptatif |