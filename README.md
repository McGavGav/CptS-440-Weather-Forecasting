**Project**

- **Description:** Weather forecasting experiments using 2002 GSOD observations. This repository builds a Static Bayesian Network (BN), a 2-time-slice Dynamic Bayesian Network (DBN), and a Latent-state DBN (HMM+DBN) to forecast binary weather events and discretized atmospheric variables.

**Requirements**

- **Python:** 3.8+ recommended. Create a venv before installing packages.
- **Key packages:** pandas, numpy, scikit-learn, pgmpy, hmmlearn, seaborn, matplotlib

Install in a fresh venv (Windows ex):

python -m venv venv
venv\Scripts\activate
pip install pandas numpy scikit-learn pgmpy hmmlearn seaborn matplotlib
pip install jupyterlab

**Data**

**NOAA Global Surface Summary of Day (GSOD)**

- Source: NOAA FTP — `ftp.ncdc.noaa.gov/pub/data/gsod/`
- Reference: [Kaggle NOAA GSOD Dataset](https://www.kaggle.com/datasets/noaa/gsod)
- Year used: **2002** 
- Key fields: `TEMP`, `DEWP`, `SLP`, `WDSP`, `PRCP`, and fog / rain / snow / thunder ina csv file
- **For graders, this csv file will be included in the folder on google. It is too large for git upload.**

**Notebook: usage**

- Open the main analysis notebook: [notebooks/BayesianNetwork.ipynb](notebooks/BayesianNetwork.ipynb) in Jupyter Lab or Notebook.
- Run the cells in order. Key sections:
	- Discretization & static BN 
	- DBN preparation, structure & training
	- HMM latent regime inference and latent DBN

**Interactive Model**

- At the bottom of the notebook there is an "Interactive Model" and a small tester code cell that lets you:
	- choose `selected_model` from `static`, `dbn`, `latent_dbn`;
	- set `user_evidence` (model dependent);
	- set `query_vars` to request posterior distributions for chosen variables.

- Usage notes:
	- Run the model building cells above first so the notebook kernel has the trained inference objects loaded.
	- Typical evidence keys:
		- Static BN: `SLP_D`, `TEMP_D`, `WIND_D`, `DEWP_D`, `PRCP_D` (same-day observations)
		- DBN / Latent DBN: same keys with `_prev` suffix (e.g. `TEMP_D_prev`) plus binary `_prev` flags like `RAIN_prev`, `SNOW_prev`, `FOG_prev`, `THUNDER_prev`.

**Testing & Evaluation**

- The notebook prints classification reports, confusion matrices, accuracy for discretized atmospheric targets, Brier score and Brier skill scores, and a macro-F1 comparison pivot for all models.
- Key evaluation cells and variables:
	- Static BN evaluation: uses `eval_df`, `predictions` and prints `classification_report` for `RAIN`, `SNOW`, `FOG`.
	- DBN evaluation: uses `dbn_eval`, `dbn_preds`, and `DBN_TARGET_COLS`.
	- Latent DBN evaluation: uses `latent_test`, `latent_preds`, and `LATENT_TARGET_COLS`.
	- Probabilistic quality: Brier scores computed in the DBN section.

**Tips & Troubleshooting**

- If the interactive tester prints that no models are loaded, re-run the notebook cells that build/train the model(s) so the kernel has `infer`, `dbn_infer`, or `dbn_latent_infer` in memory.
- If inference runs out of memory or is very slow, reduce the evaluation sample sizes (these are already limited in the notebook to keep runtime reasonable).
