# HAR Continual Learning

**Adaptive and Continual Learning for Human Activity Recognition using IMU Sensors**

---

## Protocole PFE strict (HAPT)

Le nouveau parcours séparant les sujets, conservant les sessions et évaluant une fenêtre réellement future est décrit dans [le protocole HAPT](docs/protocole_hapt_strict.md). Son entraînement reste à valider avec PyTorch ; les anciens résultats ne constituent pas sa validation.

## Overview

A two-headed deep learning system for **Human Activity Recognition (HAR)** from wearable IMU sensors that:

1. **Recognizes activities continually** — adapts to new users, new activities, and new contexts while aiming to limit forgetting (requires measured validation)
2. **Anticipates future activities** — predicts the next activity from partial/ambiguous signal windows (postural-transition prediction is not validated fall prevention)

### Architecture

```
IMU Signal (acc + gyro, 50 Hz)
        │
  Preprocessing (Butterworth, resampling, 3s windows)
        │
  Transformer Backbone  (4 blocks, 128-dim, ~531K params)
        │
  ┌─────┴──────────────────────────────────┐
  │                                        │
  ▼                                        ▼
Module 1: Continual HAR              Module 2: Anticipation
  Prototype Memory                     LSTM decoder
  + Experience Replay                  → predict next activity
  + Contrastive Loss                     from partial window
  → nearest-mean classifier
```

**Anti-catastrophic-forgetting stack:**
- Experience Replay (reservoir sampling, class-balanced)
- Prototype Memory (online running-mean class centroids)
- Supervised Contrastive Loss (inter-class separation)

---

## Project Structure

```
har_project/
├── src/
│   ├── data/
│   │   ├── preprocessing.py      # Resample, Butterworth, sliding windows
│   │   ├── dataset_loaders.py    # HAPT, MobiAct, PAMAP2, WISDM loaders
│   │   ├── homogenization.py     # Label unification, unified pipeline
│   │   └── har_dataset.py        # Dataset class + CL task sequences
│   ├── models/
│   │   ├── backbone.py           # IMUTransformerEncoder
│   │   ├── prototype_memory.py   # PrototypeMemory + contrastive loss
│   │   ├── replay_buffer.py      # Experience replay (reservoir sampling)
│   │   └── har_model.py          # HARContinualModel (full system)
│   ├── training/
│   │   └── trainer.py            # pretrain() + continual_train()
│   └── evaluation/
│       └── metrics.py            # F1, BWT, FWT, Forgetting, Intransigence
└── scripts/
    ├── preprocess.py             # Raw → processed numpy arrays
    └── train.py                  # Pre-training + continual training
```

---

## Datasets

Four public IMU datasets, homogenized to a unified format (50 Hz, m/s², 3s windows):

| Dataset | Activities | Subjects | Hz |
|---------|-----------|----------|----|
| [HAPT](https://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones) | 6 ADLs + transitions | 30 | 50 |
| [MobiAct](https://bmi.hmu.gr/the-mobiact-dataset-v2-0/) | 9 ADLs + falls | 57 | 87 |
| [PAMAP2](https://archive.ics.uci.edu/ml/datasets/PAMAP2+Physical+Activity+Monitoring) | 18 activities | 9 | 100 |
| [WISDM](https://www.cis.fordham.edu/wisdm/dataset.php) | 6 ADLs (legacy AR v1.1) | 36 in the full release; verify loaded subset | 20 |

---

## Quickstart

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Place datasets under data/raw/<name>/
#    See: python scripts/preprocess.py --help

# 3. Preprocess
python scripts/preprocess.py --data_root data/raw --out data/processed

# 4. Pre-train on merged dataset
python scripts/train.py --mode pretrain --data data/processed --epochs 50

# 5. Continual learning (user-incremental)
python scripts/train.py --mode continual --scenario user --data data/processed \
       --checkpoint checkpoints/pretrained.pt

# 6. Continual learning (class-incremental, 6 classes/task recommended)
python scripts/train.py --mode continual --scenario class --data data/processed \
       --classes_per_task 6

# 7. Démo PFE (interface graphique Streamlit)
streamlit run app_streamlit.py
```

---

## Evaluation Metrics

| Metric | Description |
|--------|-------------|
| Macro F1 | Primary recognition metric (handles class imbalance) |
| BWT (Backward Transfer) | How much old tasks degrade after learning new ones |
| Forgetting | Max accuracy drop per old task |
| FWT (Forward Transfer) | How much earlier learning helps new tasks |
| Anticipation Accuracy | Prediction accuracy at 25%, 50%, 75% observation |

---

## Key References

- Adaimi & Thomaz (2022) — *Lifelong Adaptive ML for HAR using Prototypical Networks* — LAPNet-HAR
- Amrani et al. (2025) — *Leveraging Dataset Integration and Continual Learning for HAR*
- Schiemer et al. (2023) — *Online Continual Learning for HAR* — OCL-HAR
- Kirkpatrick et al. (2017) — *Overcoming Catastrophic Forgetting in Neural Networks* — EWC
- Dirgová et al. (2022) — *Wearable Sensor-Based HAR with Transformer*

---

## License

MIT

## PFE audit — 2026-09-08

See [the audit](docs/audit_pfe_2026-09-08.md) before interpreting existing figures.
The checked-out code and the earlier manuscript differ: UWR currently uses
single-pass predictive entropy, not Monte Carlo Dropout. Regenerate processed
arrays after the preprocessing fixes; old checkpoints/results are not comparable
without rerunning the same corrected protocol. Normalization on the full corpus
is disabled. Any scaler must be fitted on training data only.

Run lightweight regression tests with `python -m unittest discover -s tests -v`.
Use `--uncertainty_replay` in `scripts/train.py` to explicitly enable UWR.
