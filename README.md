# Traders Behavior Analysis (`Traders-Behavior-Analysis`)

Quantitative analysis of **211,224 Hyperliquid executions** across **32 trader accounts** and **246 markets** (`2023-04-30` to `2025-05-01` UTC) against daily **Bitcoin Fear & Greed Index** market regimes (`2,644` daily observations, `2018-02-01` to `2025-05-02`).

---

## Key Verified Findings

- **Temporal Alignment Matters:** Raw Hyperliquid timestamps were recorded in `IST` (`Asia/Kolkata`). Converting cleanly to `UTC` shifted calendar dates for **53,322 trades (25.24%)**, eliminating **6** boundary-date mismatches and achieving **100.0% (`211,224 / 211,224`)** exact many-to-one sentiment coverage.
- **Scale & Profitability:** Across **476 active UTC days**, total notional volume reached **USD 1.191B** (`USD 1,191.19M`) with **+USD 10.30M** gross closed PnL (**+USD 10.05M** after **USD 245.9K** fees) and an **83.20%** closing-trade win rate (`104,408` closing fills).
- **FDR-Validated Regime Shifts:** **17/19** 5-regime tests (`Kruskal-Wallis H` / `Chi-Square`) and **16/19** `Fear` vs `Greed` tests remain significant after Benjamini-Hochberg FDR adjustment (`p < 0.05`). In `Fear`, perp `Open Long` share surges from **43.95% -> 61.02%** (`+17.07 pp`, 95% CI `[+16.34, +17.83]` pp), median daily volume rises (**USD 711.4K vs 271.0K**), and closing win rate increases (`+1.13 pp`).
- **Three Distinct Trader Archetypes (`KMeans`, `k = 3`):**
  - `C0: Taker-Heavy` (`n = 24`): `USD 522.2M` vol, `+USD 7.83M` net PnL (**USD 13.05K** per `1M` vol).
  - `C1: Taker-Heavy Alpha Contrarian` (`n = 5`): `USD 9.6M` vol, `+USD 1.20M` net PnL (**USD 121.35K** per `1M` vol, `94.0%` win rate, `0.384` IR, `+18.24 pp` Fear contrarian shift).
  - `C2: High-Vol Taker-Heavy Low-Margin` (`n = 3`): **USD 659.4M (`55.4%` of volume)** at `88.1%` taker execution for `+USD 1.03M` net PnL (**USD 1.53K** per `1M` vol).
- **Contrarian vs Momentum Risk Separation:** Contrarian-tilted traders (`>= 60%` contrarian opens, `n = 13`) compounded **+USD 6.03M** net PnL with **-USD 51.6K** max drawdown vs **+USD 0.52M** and **-USD 460.7K** max drawdown for the Momentum-tilted cohort (`< 40%`, `n = 11`).
- **Regime Conditioning Check:** Under zero-leakage 5-fold `TimeSeriesSplit` CV (`352` days), linear `LogisticRegression` achieved `ROC-AUC` **0.579** (high-volume days) and **0.582** (profitable days), outperforming `RandomForest` (`0.503` / `0.486`).

---

## Project Structure

```text
Traders-Behavior-Analysis/
├── notebook_1.ipynb        # Phases 1–4: Ingestion, IST->UTC cleaning, EDA & feature engineering
├── notebook_2.ipynb        # Phases 5–6: Statistical tests, KMeans clustering, signals & PDF report
├── csv_files/              # Raw, cleaned, daily/trader features, test & cluster CSV artifacts
├── outputs/                # 25+ saved 300-DPI diagnostic, EDA, clustering & signal PNG charts
├── docs/
│   ├── PLAN.md             # Complete phase-by-phase execution roadmap & checklists
│   └── FINDINGS.md         # Chronological quantitative findings log (Phases 0–6)
├── ds_report.pdf           # 6-page executive & technical PDF report compiled from outputs
├── requirements.txt        # Direct & key transitive Python/Jupyter dependencies
└── README.md               # Project overview, key insights & local execution guide
```

---

## Local Jupyter Setup & Reproduction

This repository runs completely locally in **JupyterLab / Jupyter Notebook** (no Google Colab dependencies).

1. **Create and activate a virtual environment** (inside `Traders-Behavior-Analysis/`):
   ```bash
   python -m venv .venv
   # Windows PowerShell:
   .venv\Scripts\Activate.ps1
   # macOS / Linux:
   source .venv/bin/activate
   ```
2. **Install all dependencies and register the kernel**:
   ```bash
   pip install -r requirements.txt
   python -m ipykernel install --user --name traders-behavior --display-name "Python (Traders-Behavior)"
   ```
3. **Run notebooks in order**:
   - Execute [`notebook_1.ipynb`](notebook_1.ipynb) end-to-end (**Phases 1–4**) to generate cleaned, daily-regime, and trader-profile CSVs + audit/EDA visuals.
   - Execute [`notebook_2.ipynb`](notebook_2.ipynb) end-to-end (**Phases 5–6**) to run FDR-corrected hypothesis tests, `KMeans` segmentation, event studies, `TimeSeriesSplit` CV, and compile [`ds_report.pdf`](ds_report.pdf).
4. **Full Documentation:**
   - Roadmap & Checklists: [`docs/PLAN.md`](docs/PLAN.md)
   - Detailed Phase-by-Phase Findings: [`docs/FINDINGS.md`](docs/FINDINGS.md)
