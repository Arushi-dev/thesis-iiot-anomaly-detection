# Corrected thesis notebooks

These notebooks are a **post-submission revision** of the analysis in the Master's thesis *Machine Learning-Based Anomaly Detection for Cybersecurity in Industrial IoT Environments* (Arcada UAS, 2026). The original notebooks in `../original/` and the submitted thesis are unchanged. This folder fixes problems found in a later review and re-runs every model with fixed random seeds.

## How to run
Open the notebooks in order and run all cells: `02` → `03` → `04` → `05`. Notebook `02` reads the raw Zeek logs from `../../data/normal_2/conn.csv` and `../../data/normal_DDoS_1/conn.csv`. Every later notebook reads `features_conn.csv` and the shared train/validation/test split it contains.

Requirements: pandas, numpy, scikit-learn, matplotlib, tensorflow, hpelm, jupyter. The results below came from TensorFlow 2.21. Other versions or hardware can shift the autoencoder figures slightly, even with seeds fixed.

## Problems found in the original analysis, and the fixes

| # | Problem | Effect | Fix |
|---|---|---|---|
| 1 | Every connection in the `normal_DDoS_1` capture was labelled as an attack, but 12,224 of its 123,536 connections are normal background traffic | About 10% of "attacks" were really normal traffic, which depressed recall | Attack = connection originating from TON_IoT attacker hosts `192.168.1.30`–`.39` (**assumption**: the provided ground-truth files cover port scanning only, not this DDoS capture) |
| 2 | The timestamp `ts` was a model feature. Normal traffic is from 2 April 2019 and DDoS traffic from 25 April 2019 | The date alone separates the classes | `ts`, IP addresses and port numbers removed from the features |
| 3 | The autoencoder was trained without random seeds | Each run gives different numbers. The thesis figure (AUC 0.9382), notebook (0.9121) and text (≈ 0.94) disagree | Seeds fixed for Python, NumPy and TensorFlow, with deterministic ops |
| 4 | Thresholds were set from percentiles of the **test** set's normal rows | Test labels were used to tune the model | Thresholds set on held-out validation normal traffic |
| 5 | The architecture in the code (64-32-64) did not match the thesis (64-32-16-32-64) | The described model was not the one trained | Code now matches the thesis architecture |
| 6 | Feature analysis matched autoencoder flags to feature rows by position, across files with different row orders | The feature-importance chart compared unrelated rows | All scores joined on a shared `row_id` |
| 7 | The model-comparison chart used hard-coded numbers from an older run | The chart did not reflect the final results | Chart built from saved results |
| 8 | ELM scaler fitted on all rows before the train/test split | Minor leakage | Scalers fitted on training data only, for all models |
| 9 | `history` one-hot encoding produced ~770 columns, mostly rare values | Sparse, noisy feature space | Top 20 `history` values plus `other`; 50 features in total, byte, packet and duration counts log-scaled |

## Results (shared test set: 26,143 connections, 22,262 attacks)

| Model | ROC-AUC | Precision | Recall | F1 | False-positive rate |
|---|---|---|---|---|---|
| Autoencoder, 95th pct threshold | 0.983 | 0.990 | 0.885 | 0.935 | 5.3% |
| Autoencoder, 90th pct threshold | 0.983 | 0.981 | 0.979 | 0.980 | 11.0% |
| Autoencoder, 85th pct threshold | 0.983 | 0.973 | 0.983 | 0.978 | 15.5% |
| Isolation Forest, trained on all traffic | 0.143 | 0.513 | 0.033 | 0.062 | 17.9% |
| Isolation Forest, normal-only, 80th pct | 0.767 | 0.911 | 0.358 | 0.514 | 20.0% |
| ELM (supervised) | 0.996 | 0.993 | 0.993 | 0.993 | 4.1% |

## Comparison with the submitted thesis

- **The main conclusion holds, and is stronger.** An autoencoder trained only on normal traffic detects DDoS connections well: ROC-AUC 0.98, against roughly 0.91–0.94 originally.
- **The threshold trade-off holds but is gradual.** Moving from the 95th to the 90th percentile raises recall from 0.89 to 0.98 and doubles the false-positive rate from 5% to 11%. The sharp jump in the thesis (recall 0.39 → ~1.0) does not appear once labels and features are corrected.
- **The Isolation Forest results in the thesis (precision 0.94, recall 0.91) cannot be reproduced.** Isolation Forest assumes anomalies are rare. Here 88% of the training traffic is attack traffic, so the forest treats attacks as normal (ROC-AUC 0.14). Trained on normal traffic only, it reaches ROC-AUC 0.77 but recall of only 15–36%.
- **Autoencoder–Isolation Forest agreement is 36%, not 94%.**
- **ELM (0.99) is consistent with the thesis.** It is supervised, so it is not a like-for-like comparison with the unsupervised models.
- **Both kinds of feature matter.** Attacks differ from normal traffic in volume features (bytes, packets) and in connection-state flags (`SH`, `S0`, history `F`). This reconciles the thesis text, which named volume features, with its own figure, which showed connection-state flags.
- **Capture-confound check: AUC 0.44.** The autoencoder does not score normal traffic from the two capture days differently, so it is not simply telling the two network environments apart.

## Remaining limitations
- Attack labels rely on the attacker-host assumption above.
- One DDoS capture and one normal capture; results may not generalise to other networks or attack types.
- No cross-validation or multi-seed variance estimate.
