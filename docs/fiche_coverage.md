# Couverture fiche PFE ↔ code

| Élément fiche | Statut | Où |
|---------------|--------|-----|
| HAR IMU + Transformer | OK | `src/models/backbone.py` |
| Apprentissage continu / forgetting | OK | UWR, replay, prototypes |
| Anticipation activité (signaux partiels) | OK | LSTM, F1 ~0.59–0.64 |
| Mémoire externe / attention | OK | prototypes + Transformer |
| Fine-tuning adaptatif | OK | pretrain → continual |
| **SSL auto-supervisé** | Ajouté | `scripts/train_ssl.py`, `src/training/ssl_pretrain.py` |
| **Foundation-style** (prétrain→adapt, échelle datasets) | Ajouté | `scripts/train_foundation_style.py` |
| **Calibration adaptative en ligne** | Ajouté | `src/models/online_calibration.py` |
| **Anticipation risque de chute / équilibre** | Ajouté | transitions 9–12, `scripts/train_fall_risk.py` |
| **RL léger** (bandit de seuil d’alerte) | Ajouté | `src/training/rl_threshold_agent.py` |
| **Préparation de repas** | Indispo. données | `src/scenarios/adl_scenarios.py` |

## Limites honnêtes
- Pas un foundation model public (données/compute insuffisants).
- RL = bandit de seuil, pas deep RL.
- Fall-risk = transitions posturales HAPT (pas classe `fall` absente du merge).
- Repas = scénario fiche uniquement tant qu’il n’y a pas de corpus cuisine.
