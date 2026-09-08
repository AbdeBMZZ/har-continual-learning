# Fusion corrige_v2 → har_project (8 sept. 2026)

## Fait
- Code corrigé de `Downloads/har_continual_learning_corrige_v2` fusionné
  (preproc, loaders, splits temporels, métriques, UWR, calibration, fall-risk).
- Ajouts : `temporal_protocol.py`, `prepare_hapt_protocol.py`,
  `run_hapt_protocol.py`, `tests/`, `docs/audit_pfe_2026-09-08.md`,
  `docs/protocole_hapt_strict.md`.
- Tests : **15 passed**.

## Nouveau protocole HAPT (strict)
```bash
python scripts/prepare_hapt_protocol.py --raw data/raw/hapt/RawData --acc-unit g --out data/hapt_strict_v1
python scripts/run_hapt_protocol.py --protocol data/hapt_strict_v1 --stage recognition --epochs 8 --device mps --out checkpoints/hapt_recognition_strict.pt
python scripts/run_hapt_protocol.py --protocol data/hapt_strict_v1 --stage anticipation --checkpoint checkpoints/hapt_recognition_strict.pt --ratio 0.5 --epochs 8 --device mps --out checkpoints/hapt_anticipation_p50_strict.pt
python scripts/run_hapt_protocol.py --protocol data/hapt_strict_v1 --stage test --checkpoint checkpoints/hapt_anticipation_p50_strict.pt --out results/hapt_anticipation_p50_test.json
```

### Résultats (run court, 8 époques, seed protocole 42)
| Étape | Métrique | Valeur |
|---|---|---|
| Recognition (val, best epoch) | macro-F1 | **0,777** (acc 0,866) |
| Anticipation p=0.5 (test) | macro-F1 | **0,423** (acc 0,790) |
| Baseline majorité (test) | macro-F1 | 0,038 |

Ces scores **remplacent** l’ancienne anticipation « 0,60–0,64 » pour le
protocole strict. Les anciens checkpoints (`anticipation.pt`, etc.) restent
des runs legacy et ne doivent plus être présentés comme validation du code
corrigé.

## Suite recommandée
- Relancer recognition/anticipation avec plus d’époques + plusieurs seeds.
- Relancer CL user/class après régénération `data/processed` via
  `scripts/preprocess.py` corrigé.
- Mettre à jour le mémoire avec ces chiffres stricts.
