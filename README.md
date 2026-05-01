# DATATHON_2026
Part 2: Link for part 2 PowerBI dashboard: https://drive.google.com/drive/folders/1PLNIlpSzZScQ_KJc4I97NOB_AYHqzAlR?usp=sharing
# Datathon 2026 — Round 1 Solution (Part 3 — Kaggle)

Forecast daily `Revenue` and `COGS` for 2023-01-01 → 2024-07-01.

See [`plan.md`](plan.md) for full design rationale.

## Quick start

```bash
pip install -r requirements.txt
# from the project root (containing sales.csv, sample_submission.csv, …)
python -m solution.src.train
```

Output: `solution/submissions/submission.csv` — ready to upload to Kaggle.

## Pipeline

1. Calendar + Vietnamese holiday + e-commerce-event features (Tết, 11/11, Black Friday, …).
2. Year-ago lag features at lag 365 / 730 / 1095 (no recursion needed for the 18-month horizon).
3. Rolling means/std anchored at lag 365 AND lag 730 (so 2024 test rows still get rolling context from lag-730).
4. **Dual LightGBM** (log-target):
   - Model A — full feature set, predicts the 2023 portion (lag-365 known).
   - Model B — drops lag-365 features, predicts the 2024 portion (lag-365 falls inside the unknown test window).
5. Time-series CV on 2021 + 2022 holdouts → picks `best_iter` per model.
6. Final retrain on 2012–2022, predict, write submission preserving sample-submission row order.

## Folder layout

```
solution/
├── plan.md              # design doc
├── README.md
├── requirements.txt
├── src/
│   ├── config.py        # paths, seed, LightGBM hyperparams
│   ├── holidays_vn.py   # hard-coded VN events 2012–2025
│   ├── features.py      # feature engineering pipeline
│   ├── models.py        # LightGBM wrapper, seasonal-naive, metrics
│   └── train.py         # entry point: CV → fit → predict → submission
├── outputs/             # CV metrics JSON + per-model feature importance CSV
└── submissions/         # submission.csv
```

## Reproducibility

- Single random seed `SEED=42` (set in `solution/src/config.py`) used by NumPy, Python `random`, LightGBM.
- No external data — every feature is derived from `sales.csv` + the hard-coded holiday calendar.
