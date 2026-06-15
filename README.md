# PTT_viability_prediction
For prediction of viability of cancer spheroids via their morphological changes upon PTT treatment using brightfield images.
# Spheroid Viability Classification via Morphological Features and Machine Learning

---

## Overview

This repository contains the code accompanying the above publication. We classify the
viability of tumour spheroids (cancer cell aggregates) after **photothermal therapy (PTT)**
using morphological and texture features extracted from brightfield images at three
time-points (day 0, 1, 2). Two supervised classifiers are trained and compared:

- **Random Forest (RF)** with stratified 5-fold cross-validation and SMOTE balancing
- **Neural Network (NN)** (TensorFlow/Keras) with early stopping, dropout, and L2 regularisation

A **Bayesian MCMC** approach (via `emcee`) estimates the IT₅₀ — the laser temperature
at which 50 % of spheroids are predicted to be non-viable.

Five breast cancer cell lines are supported: **MCF7**, **MCF10DCIS**, **MDA-MB-231**,
**Hs578T**, and a **PDX** (patient-derived xenograft) of a triple negative breast cancer model.

---

## Repository Structure

```
.
├── ptt_rf_nn_classifier.ipynb       # Main ML pipeline (RF + NN, IT50, visualisations)
├── requirements.txt               # Python dependencies
├── LICENSE                        # MIT licence
├── data/                          # ← Place your CSV files here (see Data section below)
│   ├── mcf7/
│   │   └── combined_immediate_days.csv
│   ├── mda/
│   ├── mcf10/
│   ├── hs578t/
│   └── pdx/
└── output figures/                # Auto-created by notebook to store SVG/PNG outputs
```

---

## Notebooks

### `ptt_rf_nn_classifier.ipynb` — ML Classification Pipeline

| Section | Description |
|---------|-------------|
| 0 — Imports & Config | All tunable parameters in one place (paths, days, NaN strategy, SMOTE, regularisation) |
| 1 — Data Loading | CSV ingestion, row filtering (valid label + temperature required) |
| 2 — Feature Extraction | Morphology + texture features per day; temperature excluded (leakage prevention) |
| 3 — Feature Engineering | Growth rates, grey-value changes, compactness, interaction terms |
| 4 — Feature Selection | Optional SelectKBest (mutual information); all features by default |
| 5 — RF 5-Fold CV | Stratified K-fold, per-fold metrics, out-of-fold predictions |
| 6 — Overfitting Analysis | Train vs validation accuracy gap per fold |
| 7 — Final RF Model | 75/25 holdout or cross-dataset mode; optional SMOTE |
| 8 — RF Visualisations | Confusion matrix, ROC, feature importance, precision–recall vs threshold |
| 8a — IC₅₀ (RF) | Bayesian MCMC sigmoid fit to temperature vs predicted viability |
| 8b — Neural Network | Training with early stopping, dropout, L2; performance metrics |
| 8b-ii — NN 5-Fold CV | Same folds as RF for fair comparison |
| 8b-iii — Overfitting (NN) | NN training curves; RF vs NN overfitting comparison |
| 8c–8e — NN Visualisations | Confusion matrix, ROC, training curves, IC₅₀ (NN) |
| 9–9b — Violin Plots | Temperature distribution by predicted / true label (test set + OOF) |
| 10 — CSV Export | Out-of-fold predictions saved to CSV |
| 11 —  ROC | RF + NN ROC on a single publication-ready axes |
| 12 —  Box Plots | K-fold metrics: RF only, NN only, RF vs NN side-by-side |
| Final cell — Multi-Cell-Line | Interactive comparison across all cell lines (ipywidgets) |

---

## Data

The notebooks expect per-cell-line CSV files are in the respective folders.

**Expected columns (subset):**

| Column | Description |
|--------|-------------|
| `Label_day2` | Viability class at day 2: `Alive` or `Dead` |
| `Max temp (°C)_day2` | Maximum laser-spot temperature at day 2 (°C) |
| `Area (pix2)_day0/1/2` | Spheroid area (pixels²) at each day |
| `Circularity_day0/1/2` | Circularity (0–1) |
| `Solidity_day0/1/2` | Solidity (0–1) |
| `Perimeter (pix)_day0/1/2` | Perimeter (pixels) |
| `Aspect ratio_day0/1/2` | Major/minor axis ratio |
| `Mean grey value_day0/1/2` | Mean pixel intensity |
| `Homogeneity_day0/1/2` | GLCM homogeneity |
| `Energy_day0/1/2` | GLCM energy |
| `Correlation_day0/1/2` | GLCM correlation |
| `Equivalent Diameter (pix)_day0/1/2` | Equivalent circular diameter |
| `Normalised alb` | Normalised metabolic activity (%) |


---


---

## Usage

### Quick start

```bash
jupyter lab
```

Open `ptt_rf_nn_classifier.ipynb` and update the **Configuration** cell (Section 0b):

```python
# Single cell-line mode (internal 75/25 split):
CROSS_DATASET_MODE = False
CELL_LINE_CSVS = {
    "MCF7": "data/0h NP incubation/MCF7.csv",
    # add further cell lines as needed
}

# Cross-dataset mode (train on one CSV, test on another):
CROSS_DATASET_MODE = True
TRAIN_CSV = ""data/0h NP incubation/MDAMB231.csv"
TEST_CSV  = "data/0h NP incubation/PDX.csv"
```

Then **Run All Cells** (`Kernel → Restart & Run All`).

### Key configuration options

| Parameter | Default | Description |
|-----------|---------|-------------|
| `DAYS_TO_USE` | `['day0','day1','day2']` | Which day columns to include |
| `INCLUDE_DELTA` | `False` | Add day-to-day difference columns |
| `NAN_STRATEGY` | `'interpolate'` | `'interpolate'`, `'median'`, or `'drop'` |
| `USE_FEATURE_SELECTION` | `False` | Apply SelectKBest (mutual information) |
| `N_FEATURES` | `30` | Number of top features to keep |
| `USE_SMOTE` | `True` | Balance training set with SMOTE |
| `DROPOUT_RATE` | `0.3` | NN dropout fraction (0 = disabled) |
| `L2_REG` | `0.001` | NN L2 weight regularisation |
| `EARLY_STOPPING` | `True` | Stop NN training on val_loss plateau |

---

## Output files

All figures and CSVs are saved to the same directory as the input CSV by default.

| File | Description |
|------|-------------|
| `roc_curve_rf_nn.png` | RF + NN ROC curve |
| `kfold_metrics_boxplot_RF.png` | K-fold metrics box plot (RF) |
| `kfold_metrics_boxplot_NN.png` | K-fold metrics box plot (NN) |
| `kfold_metrics_boxplot_RF_vs_NN.png` | Side-by-side RF vs NN comparison |
| `violin_temperature_labels.png` | Violin: temperature by predicted / true label (test set) |
| `violin_temperature_labels_oof.png` | Violin: temperature by OOF label (training data) |
| `predictions_export.csv` | Out-of-fold predictions (RF + NN) for every training sample |
| `output figures/*.svg` | Per-feature and viability score figures (notebook 2) |



---

## Licence

This project is licensed under the **MIT Licence** — see [`LICENSE`](LICENSE) for details.
