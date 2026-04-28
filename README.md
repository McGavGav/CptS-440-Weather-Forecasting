# CptS-440-Weather-Forecasting

**WSU CptS 440 — Bayesian Network Weather Forecasting**

## Overview

This project builds a next-day weather forecasting system using Bayesian Networks trained on historical NOAA GSOD meteorological data. The approach progresses from a static Bayesian Network (BN) to a Dynamic Bayesian Network (DBN), and then to a Latent-State DBN that incorporates hidden atmospheric regimes inferred via a Gaussian Hidden Markov Model (HMM).

Weather variables — temperature, dew point, wind speed, sea-level pressure, and precipitation — serve as observed nodes. Unobserved large-scale atmospheric patterns (pressure systems, air masses) are captured as latent hidden states. The system forecasts next-day weather events (rain, snow, fog, thunder) as well as next-day discretized atmospheric variable classes (temperature band, pressure level, wind strength).

## Data Source

**NOAA Global Surface Summary of Day (GSOD)**

- Source: NOAA FTP — `ftp.ncdc.noaa.gov/pub/data/gsod/`
- Reference: [Kaggle NOAA GSOD Dataset](https://www.kaggle.com/datasets/noaa/gsod)
- Year used: **2002** (~9,000 global weather stations)
- Key fields: `TEMP`, `DEWP`, `SLP`, `WDSP`, `PRCP`, and the `FRSHTT` flag string (fog / rain / snow / hail / thunder / tornado indicators)

## Model Progression

| Model | Description |
|---|---|
| **Static BN** | Discrete Bayesian Network with a meteorologically-motivated DAG. Trained with a Bayesian Estimator (K2/Dirichlet prior) to handle zero-count CPT cells. Provides same-day conditional inference. |
| **Dynamic BN (2-TBN)** | Extends the static BN with inter-slice temporal edges (`_prev → current`), enabling genuine **next-day forecasts**. Uses a time-ordered 80/20 train/test split to prevent look-ahead bias. |
| **Latent-State DBN** | Adds HMM-inferred atmospheric regimes as hidden nodes. A `GaussianHMM` (3 components, diagonal covariance) decodes latent weather regimes from continuous observations; the regime is integrated out at forecast time, giving a principled latent-variable structure aligned with real-world atmospheric dynamics. |

## Repository Structure

```
├── data/
│   ├── raw/gsod/2002/          # Raw per-station .op.gz files (downloaded by notebook)
│   └── processed/gsod_2002.csv # Cleaned, combined single-file dataset
├── models/                     # Saved model artifacts (future)
├── notebooks/
│   ├── 2002_Data_Grabber.ipynb # Data download, extraction, parsing, and cleaning
│   └── BayesianNetwork.ipynb   # Static BN → DBN → Latent-State DBN (full pipeline)
├── reports/                    # Figures and evaluation outputs (future)
└── README.md
```

## Setup

### Requirements

```bash
pip install pgmpy hmmlearn pandas numpy scikit-learn matplotlib seaborn pyarrow
```

The first cell of `2002_Data_Grabber.ipynb` installs all dependencies automatically via `%pip install`.

### 1 — Download and Process the Data

Open and run all cells in `notebooks/2002_Data_Grabber.ipynb`. This will:

1. Connect to the NOAA FTP server (anonymous login — no credentials required)
2. Download `gsod_2002.tar` (~25 MB)
3. Extract ~9,000 per-station `.op.gz` files into `data/raw/gsod/2002/`
4. Parse NOAA fixed-width format, apply missing-value sentinels, decode FRSHTT flags
5. Write the cleaned dataset to `data/processed/gsod_2002.csv`

### 2 — Run the Forecasting Models

Open and run all cells in `notebooks/BayesianNetwork.ipynb` top-to-bottom. The notebook is fully self-contained and will:

- Discretize continuous variables into meteorologically-meaningful bins
- Build and train the Static BN, DBN, and Latent-State DBN
- Run scenario-based inference for each model
- Evaluate all models on a held-out test set (time-ordered for DBN/Latent DBN)
- Display classification reports, confusion matrices, Brier scores, and a model comparison summary

## Evaluation

All models are evaluated on four binary weather events: **RAIN**, **SNOW**, **FOG**, **THUNDER**; and the DBN additionally forecasts five discretized atmospheric variables: **TEMP_D**, **SLP_D**, **WIND_D**, **DEWP_D**, **PRCP_D**.

| Model | Split | Eval Sample |
|---|---|---|
| Static BN | Random 80/20 | 2,000 rows |
| DBN | Time-ordered 80/20 | 2,000 rows |
| Latent DBN | Time-ordered 80/20 | 500 rows (exact inference runtime cost) |
| Persistence Baseline | DBN hold-out | 2,000 rows |

**Metrics reported:** accuracy, macro-F1, per-class precision/recall, confusion matrices, Brier score (probabilistic calibration), and Brier Skill Score vs. climatological baseline.

## References

1. [NOAA GSOD on Kaggle](https://www.kaggle.com/datasets/noaa/gsod)
2. [pgmpy documentation](https://pgmpy.org)
3. [hmmlearn documentation](https://hmmlearn.readthedocs.io)
