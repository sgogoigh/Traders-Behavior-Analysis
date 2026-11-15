# Traders Behavior Analysis — Findings Log

> **Project:** `Traders-Behavior-Analysis`  
> **Companion Plan:** [`docs/PLAN.md`](PLAN.md)  
> **Note:** Findings are appended sequentially by phase and section strictly from inspected files, executed notebook outputs, exported CSVs, and generated visuals.

---

## Phase 0: Environment & Repository Initialization

- **Status:** Completed
- **Summary:**
  - Initialized repository directory structure: `csv_files/`, `outputs/`, `docs/`.
  - Created local Jupyter notebooks [`notebook_1.ipynb`](../notebook_1.ipynb) and [`notebook_2.ipynb`](../notebook_2.ipynb) with relative `pathlib.Path` configuration and artifact helpers (`save_fig`, `save_table_image`, `export_csv`).
  - Configured local `.venv` (Python 3.12.10) with [`requirements.txt`](../requirements.txt), [`docs/PLAN.md`](PLAN.md), and [`README.md`](../README.md).

---

## Phase 1: Data Acquisition & Initial Schema Inspection

- **Status:** Completed
- **Artifacts Generated:**
  - [`csv_files/00_data_audit_summary.csv`](../csv_files/00_data_audit_summary.csv)
  - [`outputs/01_audit_schema_summary.png`](../outputs/01_audit_schema_summary.png)
  - [`outputs/01_audit_missingness.png`](../outputs/01_audit_missingness.png)
- **1.1 Dataset Dimensions & Integrity:**
  - `fear_greed_index.csv`: **2,644 rows × 4 columns** (`0.35 MB`), **0 missing cells**, **0 exact duplicate rows**, covering `2018-02-01` to `2025-05-02`. Contains `90` distinct index values across `5` sentiment classifications.
  - `historical_data.csv`: **211,224 rows × 16 columns** (`107.16 MB`), **0 missing cells**, **0 exact duplicate rows**. Contains **32 unique `Account`s**, **246 unique `Coin`s**, **12 `Direction` types**, **2 `Side` values**, **101,184 unique `Transaction Hash`es**, **50,555 unique `Order ID`s**, and **27,977 unique minute-level `Timestamp IST`s**.
- **1.2 Schema & Export Quality Observations:**
  - **Lossy scientific-notation exports:** `Timestamp` has only **7 unique values** and `Trade ID` has only **2,810 unique values** across 211,224 rows due to scientific-notation truncation (`1.73E+12`, `8.95E+14`). `Timestamp IST` retains exact minute-level resolution (`DD-MM-YYYY HH:MM`) and was selected as the authoritative execution time.
  - **Partial-fill granularity:** Multiple distinct fills share the same `Order ID` (50,555 unique) and `Transaction Hash` (101,184 unique) with distinct prices, token sizes, and starting positions.
  - **Schema scope:** No `Leverage` column exists in `historical_data.csv`; risk/execution behavior is captured via `Size USD`, `Start Position`, `Fee`, `Crossed` (maker/taker), `Direction`, and `Closed PnL`.

---

## Phase 2: Data Cleaning, Normalization & Temporal Alignment

- **Status:** Completed
- **Artifacts Generated:**
  - [`csv_files/01_clean_sentiment.csv`](../csv_files/01_clean_sentiment.csv) (`2,644 × 12`)
  - [`csv_files/01_clean_trader_data.csv`](../csv_files/01_clean_trader_data.csv) (`211,224 × 32`)
  - [`csv_files/02_merged_trader_sentiment.csv`](../csv_files/02_merged_trader_sentiment.csv) (`211,224 × 39`)
  - [`csv_files/02_phase2_cleaning_summary.csv`](../csv_files/02_phase2_cleaning_summary.csv)
  - [`outputs/01_audit_phase2_summary_table.png`](../outputs/01_audit_phase2_summary_table.png)
  - [`outputs/01_audit_cleaning_funnel.png`](../outputs/01_audit_cleaning_funnel.png)
  - [`outputs/01_audit_temporal_coverage.png`](../outputs/01_audit_temporal_coverage.png)
- **2.1 Sentiment Series Audit (`fear_greed_index`):**
  - Zero duplicate dates; **4 missing calendar days** in the full 2018–2025 span (`2018-04-14`, `2018-04-15`, `2018-04-16`, `2024-10-26`).
  - Full-series regime counts: `Fear` (**781**), `Greed` (**633**), `Extreme Fear` (**508**), `Neutral` (**396**), `Extreme Greed` (**326**).
- **2.2 Trader Normalization & Timestamp Alignment (`historical_data`):**
  - **0 numeric coercion failures** and **0 timestamp parse failures** across all 211,224 rows.
  - **100.00% row retention** (`211,224` raw → `211,224` clean; `0` exact duplicates, `0` critical nulls).
  - **UTC date window:** `2023-04-30` to `2025-05-01`.
  - **Critical timezone finding:** **53,322 trades (25.24%)** shift calendar date when converting from `IST` (`Asia/Kolkata`) to `UTC`. Joining on `trade_date_utc` achieved **211,224 / 211,224 (100.00%, 0 unmatched)** coverage, whereas joining on raw `IST` dates would have left **6 unmatched trades** (boundary / missing-date mismatch).
- **2.3 Execution Mix & Non-Destructive Diagnostic Flags:**
  - **Direction breakdown:** `Open Long` (**49,895**), `Close Long` (**48,678**), `Open Short` (**39,741**), `Close Short` (**36,013**), `Sell` (**19,902**), `Buy` (**16,716**), `Spot Dust Conversion` (**142**), `Short > Long` (**70**), `Long > Short` (**57**), `Auto-Deleveraging` (**8**), `Liquidated Isolated Short` (**1**), `Settlement` (**1**).
  - **Closing / Realized-PnL trades (`Closed PnL != 0`):** **104,408 (49.43%)**.
  - **Execution role:** **82,821 maker trades** (`Crossed=False`, **39.21%**) vs **128,403 taker trades** (`Crossed=True`, **60.79%**), with **2,476 negative-fee trades** (maker rebates).
  - **Asset mix:** **33,986 spot index trades** (`@coin`, **16.09%**) vs **177,238 standard/perp trades** (**83.91%**).
  - **Consistency & tail flags:** **0 notional mismatches** (`size_usd` matches `execution_price * size_tokens` across all trades); **212 extreme-size fills** (`> P99.9`) preserved for analysis.
- **2.4 Temporal Coverage Observation:**
  - From `01_audit_temporal_coverage.png`, early trading activity appears sparsely around `2023-05`, followed by a multi-month gap before sustained daily trading begins in `2023-12` and accelerates sharply through late `2024` and `2025`.

---

## Phase 3: Exploratory Data Analysis (EDA)

- **Status:** Completed
- **Artifacts Generated:**
  - CSVs: [`03a_eda_sentiment_summary.csv`](../csv_files/03a_eda_sentiment_summary.csv), [`03b_eda_global_kpis.csv`](../csv_files/03b_eda_global_kpis.csv), [`03b_eda_top_coins.csv`](../csv_files/03b_eda_top_coins.csv), [`03b_eda_account_overview.csv`](../csv_files/03b_eda_account_overview.csv), [`03c_eda_regime_summary.csv`](../csv_files/03c_eda_regime_summary.csv), [`03d_eda_correlations.csv`](../csv_files/03d_eda_correlations.csv), [`03d_eda_lead_lag.csv`](../csv_files/03d_eda_lead_lag.csv)
  - Images: [`02_eda_sentiment_timeline.png`](../outputs/02_eda_sentiment_timeline.png), [`02_eda_sentiment_transitions.png`](../outputs/02_eda_sentiment_transitions.png), [`03_eda_global_kpis_table.png`](../outputs/03_eda_global_kpis_table.png), [`03_eda_pnl_distribution.png`](../outputs/03_eda_pnl_distribution.png), [`03_eda_volume_distribution.png`](../outputs/03_eda_volume_distribution.png), [`03_eda_execution_and_side.png`](../outputs/03_eda_execution_and_side.png), [`03_eda_symbol_breakdown.png`](../outputs/03_eda_symbol_breakdown.png), [`03_eda_trader_concentration.png`](../outputs/03_eda_trader_concentration.png), [`04_eda_regime_summary_table.png`](../outputs/04_eda_regime_summary_table.png), [`04_eda_activity_by_regime.png`](../outputs/04_eda_activity_by_regime.png), [`04_eda_pnl_by_regime.png`](../outputs/04_eda_pnl_by_regime.png), [`04_eda_risk_and_bias_by_regime.png`](../outputs/04_eda_risk_and_bias_by_regime.png), [`04_eda_temporal_heatmap_by_regime.png`](../outputs/04_eda_temporal_heatmap_by_regime.png), [`05_eda_correlation_heatmap.png`](../outputs/05_eda_correlation_heatmap.png), [`05_eda_lead_lag_analysis.png`](../outputs/05_eda_lead_lag_analysis.png)

### 3A. Market Sentiment / Risk Regimes
- **Full History vs Active Trading Window Skew:** Across 2018–2025 (`2,644` days), Fear regimes dominate (**48.75%**: `Fear` `29.54%`, `Extreme Fear` `19.21%`) vs Greed (**36.27%**). During the **476 active UTC trading days** (`2023-04-30` to `2025-05-01`), the market skewed strongly bullish/Greed: `Greed` **41.39%** (`197` days) + `Extreme Greed` **23.53%** (`112` days) = **64.92% Greed**, vs `Fear` **18.70%** (`89` days) and `Extreme Fear` **2.94%** (`14` days).
- **Regime Persistence:** Next-day self-transition probabilities are high (`Extreme Greed` **83.1%**, `Extreme Fear` **82.1%**, `Greed` **78.8%**, `Fear` **78.1%**), whereas `Neutral` is the shortest/most transient bridge state (**64.1%** self-transition; mean streak **2.79 days** vs **5.93 days** for `Extreme Greed` [max `77d`] and **5.52 days** for `Extreme Fear` [max `74d`]).

### 3B. Global Trader Activity, Volume, Concentration & Profitability
- **Aggregate Scale:** **211,224 trades** across **32 accounts**, **246 coins**, and **476 active UTC days**, generating **$1,191,187,442.46** in notional volume.
- **Heavy-Tailed Trade Sizing:** Median trade size is **$597.05** vs mean **$5,639.45** (`9.4×` median) and P95 **$20,023.01**.
- **Cohort Profitability:** **29 of 32 accounts (90.6%)** are net profitable before fees (`28 of 32` net positive after fees). Aggregate gross profit = **$13,245,745.87**, gross loss = **-$2,948,786.93** (**Profit Factor = 4.492**), closing-trade win rate = **83.20%** (`104,408` closing fills), net closed PnL = **$10,296,958.94** (**$10,051,101.22** after **$245,857.72** fees).
- **Whale & Asset Concentration:** Volume Gini across the 32 accounts = **0.708** (Trade Count Gini = **0.570**). Top volume account `0x513b...4ff1` drove **$420.88M** (**35.3%** of cohort volume), while top PnL leader `0xb123...ed23` earned **+$2.14M** (`0x0833...9012` **+$1.60M**). Top coins by volume: `BTC` (**$644.23M**, `+$868.0K` PnL), `HYPE` (**$141.99M**, `+$1.95M`), `SOL` (**$125.07M**, `+$1.64M`), `ETH` (**$118.28M**, `+$1.32M`), and `@107` (**$55.76M**, **+$2.78M** — #1 PnL asset). Largest loss assets: `TRUMP` (**-$364.8K**) and `FARTCOIN` (**-$100.7K**).

### 3C. Trader Behavior & Performance Across Sentiment Regimes
- **Fear Triggers Dramatic Activity Surges:** Daily-normalized trading intensity peaks in **`Extreme Fear`**: **1,521.64 trades/day**, **$8.316M/day**, **11.00 active accounts/day**, and **$76,595.28/day** net PnL — more than `6×` the daily trade rate and `6×` the daily volume of `Greed` (**247.05 trades/day**, **$1.368M/day**, **3.34 accounts/day**).
- **Monotonic Position-Size Scaling:** Median trade size increases steadily as sentiment cools: `Extreme Greed` (**$500.20**) → `Greed` (**$555.00**) → `Neutral` (**$556.81**) → `Fear` (**$699.25**) → `Extreme Fear` (**$789.07**), with P90 trade size peaking in `Fear` (**$11,015.35**).
- **Strong Contrarian Directional Bias:** On perpetual opens, `Open Long` share is **65.28%** in `Extreme Fear`, **59.62%** in `Fear`, and **65.69%** in `Neutral`, flipping to **Short-biased** in `Greed` (**41.59%** Open Long) and `Extreme Greed` (**47.17%** Open Long).
- **Profitability Efficiency vs Volume Paradox:** `Extreme Fear` delivers the highest daily aggregate PnL (**$76.6K/day**) via sheer volume, yet exhibits the **lowest win rate (78.45%)** and **lowest profit factor (2.846)**. Conversely, **`Extreme Greed`** delivers the **highest win rate (88.91%)**, **highest mean PnL per closing trade ($124.92)**, and a **10.261 profit factor**, alongside the highest **spot-index share (26.94% vs 7.71% in Extreme Fear)** and **taker share (62.50% vs 59.47%)**.

### 3D. Daily Correlations & Calendar Lead-Lag Dynamics
- **Nonlinear Fear-Spike Effect:** Daily `sentiment_value` correlates negatively with `active_accounts` (Pearson `r = -0.286`, Spearman `r = -0.166`), `total_volume_usd` (Pearson `-0.274`, Spearman `-0.071`), `total_fees` (Pearson `-0.268`, Spearman `-0.198`), and `trade_count` (Pearson `-0.251`, Spearman `-0.050`). The larger Pearson vs Spearman magnitude confirms activity spikes concentrate in tail Fear episodes rather than shifting smoothly across moderate days.
- **Lead-Lag Signature:** Across calendar lags `k ∈ [-5, +5]`, correlations remain modest (`|r| <= 0.115`). `buy_vol_pct` shows slight negative correlation at `k = -2` (`-0.113`) and `k = -1` (`-0.110`) before flattening near `0` at `k >= 0`, while `taker_pct` stays mildly negative (`-0.099` to `-0.115`) for `k = 0..+5`.

---

## Phase 4: Feature Engineering & Behavioral/Risk Metrics

- **Status:** Code Ready in `notebook_1.ipynb` (awaiting user execution in Jupyter)
- **Planned Outputs:**
  - `csv_files/03_features_daily_regime.csv` (daily regime, rolling volume/PnL, dispersion, contrarian share)
  - `csv_files/04_features_trader_profiles.csv` (32-account performance, drawdown/IR, taker/spot mix, Fear-vs-Greed behavior & contrarian bias shift)
  - `outputs/06_features_distributions.png`, `outputs/06_features_summary_table.png`, `outputs/06_features_fear_vs_greed_trader_scatter.png`

---

## Phase 5: Statistical Testing, Trader Segmentation & Signal Discovery

- **Status:** Not Started

---

## Phase 6: Insights Synthesis & Final Report (`ds_report.pdf`)

- **Status:** Not Started
