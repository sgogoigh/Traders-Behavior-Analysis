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

- **Status:** Completed (executed locally by user in `notebook_1.ipynb` Section 4)
- **Generated Datasets:**
  - `csv_files/03_features_daily_regime.csv`: **476 active UTC days x 43 columns** (`2023-04-30` to `2025-05-01`), combining daily sentiment lags/moving averages/volatility/streak indicators with daily activity, sizing, execution (`taker_pct`, `spot_pct`, `avg_fee_bps`), PnL efficiency, directional/contrarian open counts, and 7-day rolling windows.
  - `csv_files/04_features_trader_profiles.csv`: **32 trader accounts x 63 columns** covering total activity, active/calendar span, sizing dispersion (`median_size_usd`, `p90_size_usd`, `size_cv`), execution mix, closing PnL & fee net-efficiency, daily PnL stability (`daily_pnl_ir`, `max_daily_drawdown_usd`, `profitable_days_pct`), directional/contrarian share, top-coin concentration, and Fear-vs-Greed regime splits.
- **Verified Summary Statistics Across All 32 Accounts (`csv_files/04_features_trader_profiles.csv`, `outputs/06_features_summary_table.png`):**
  - **Participation & Volume:** Every account has at least **332 trades** (`min = 332`, `median = 3,699`, `mean = 6,600.75`, `max = 40,184`), so all 32 accounts remain eligible for Phase 5 segmentation. `total_volume_usd` averages **\$37.22M** (median **\$11.74M**, min **\$728.06K**, max **\$420.88M**).
  - **Sizing & Dispersion:** `median_size_usd` averages **\$1,189.71** (median **\$658.98**, min **\$148.31**, max **\$7,755.14**), with `size_cv` averaging **3.00** (median **3.06**, range **1.30–5.67**).
  - **Execution Style (Bimodal):** `taker_pct` averages **51.44%** (median **58.12%**, min **0.28%**, max **100.00%**), with `outputs/06_features_distributions.png` showing a clear split between maker-focused accounts (`< 25%` taker) and taker-heavy accounts (`> 70%` taker). `spot_pct` averages **17.89%** (median **6.98%**, range **0.00%–72.68%**).
  - **Risk-Adjusted Performance & Efficiency:** `win_rate_pct` averages **85.01%** (median **84.20%**, min **59.90%**, max **100.00%**); `pnl_per_1m_volume` averages **\$28,889.93** (median **\$12,666.87**, range **-\$22,131.75** to **+\$149,344.27**); `daily_pnl_ir` averages **0.31** (median **0.31**, range **-0.16** to **0.68**); `max_daily_drawdown_usd` averages **-\$51,042.47** (median **-\$16,766.61**, worst **-\$369,393.23**, best **\$0.00**).
  - **Contrarian Orientation & Regime Shifts:** `contrarian_open_share_pct` averages **51.21%** (median **54.88%**, range **15.85%–99.97%**). Among accounts active in both Fear and Greed (`count = 31`), `contrarian_bias_shift` (Fear minus Greed contrarian share) averages **-1.00 pp** (median **0.00 pp**, range **-100.00 pp** to **+53.42 pp**).
- **Edge Cases / Feature Hygiene Notes for Phase 5 Clustering:**
  - `profit_factor` is `NaN` for **4 accounts** (`count = 28`) that recorded **\$0 gross loss** (`100%` closing win rate: `0x3f9a...6cf6`, `0x420a...4641`, `0x430f...7713`, `0xae5e...82ed`).
  - `contrarian_bias_shift` (and Greed splits) are `NaN` for **1 account** (`count = 31`: `0xb899...2115`), which executed **4,532 Fear trades** and **0 Greed trades**. Phase 5 clustering will therefore use complete `32/32` non-null metrics (`win_rate_pct`, `daily_pnl_ir`, `pnl_per_1m_volume`, `contrarian_open_share_pct`, etc.) for scaling/KMeans and analyze `profit_factor` and `contrarian_bias_shift` post-hoc per cluster.
- **Visual Outputs Inspected:**
  - `outputs/06_features_summary_table.png`, `outputs/06_features_distributions.png`, `outputs/06_features_fear_vs_greed_trader_scatter.png`

---

## Phase 5: Statistical Testing, Trader Segmentation & Signal Discovery

- **Status:** Completed (executed locally by user in `notebook_2.ipynb` Sections 1–4)

### 5A. Hypothesis Testing & Bootstrap Confidence Intervals (`05_stats_regime_tests.csv`, `05_stats_bootstrap_ci.csv`)
- **Multiple-Testing Summary:** Out of **19 continuous & contingency tests** across the 5 sentiment regimes (`Kruskal-Wallis H` / `Chi-Square`) and coarse `Fear` vs `Greed` (`Mann-Whitney U` / `Chi-Square`), **17/19** 5-regime tests and **16/19** `Fear` vs `Greed` pairwise tests remain statistically significant after Benjamini-Hochberg FDR adjustment (`p_fdr < 0.05`).
- **Validated Daily-Level Regime Differences (`Fear n = 103 days` vs `Greed n = 309 days`, medians):**
  - **Daily Spot Share:** **8.36%** in Fear vs **0.00%** in Greed (`Cliff's Delta = +0.512`, FDR `p = 3.91e-17`) — *Simpson/aggregation distinction:* trade-weighted spot share is conversely **10.04%** in Fear vs **23.51%** in Greed (`Cramér's V = 0.179`, FDR `p ≈ 0`) because high-frequency spot accounts concentrate bursts on Greed days.
  - **Daily Active Accounts & Trade Count:** **8.0** vs **2.0** active accounts/day (`Cliff's Delta = +0.385`, FDR `p = 4.85e-09`) and **269** vs **61** trades/day (`Cliff's Delta = +0.227`, FDR `p = 9.57e-04`).
  - **Daily Volume & Net PnL After Fees:** **\$711,405.40** vs **\$270,989.97** median daily volume (`Cliff's Delta = +0.196`, FDR `p = 4.61e-03`; 2,000-resample Bootstrap 95% CI diff = **+\$0.44M**, `[+\$0.02M, +\$2.48M]`, excludes 0) and **\$4,057.19** vs **\$1,070.27** median daily net PnL after fees (`Cliff's Delta = +0.158`, FDR `p = 2.26e-02`; mean daily gross PnL diff = **+\$21.90K**, 95% CI `[+\$3.90K, +\$41.51K]`).
  - **Null / Non-Significant Daily Shifts:** Daily `win_rate_pct` (`94.08%` vs `96.07%`, FDR `p = 0.432`), daily `taker_pct` (`76.32%` vs `85.14%`, FDR `p = 0.619`), and daily `open_long_share_pct` (`67.94%` vs `51.15%`, FDR `p = 0.085`) do not reach `p_fdr < 0.05` pairwise at the unweighted day level.
- **Validated Trade-Level Regime Shifts (`N = 211,224` trades; `N = 104,408` closing; `N = 89,636` perp opens):**
  - **Perp Directional Contrarianism (`Open Long` Share):** **61.02%** in Fear (`n = 37,522`) vs **43.95%** in Greed (`n = 33,281`) — a **+17.07 pp** jump (1,000-resample Bootstrap 95% CI `[+16.34 pp, +17.83 pp]`, `Cramér's V = 0.171`, FDR `p ≈ 0`).
  - **Closing Win Rate & Notional Size:** Win rate is **84.44%** in Fear vs **83.31%** in Greed (**+1.13 pp**, 95% CI `[+0.65 pp, +1.61 pp]`, `Cramér's V = 0.015`, FDR `p = 1.39e-05`), and median trade size is **\$730.83** vs **\$550.02** (**+\$180.81**, 95% CI `[+\$163.12, +\$198.36]`, Rank-Biserial `= +0.096`, FDR `p = 3.26e-258`).
  - **Per-Dollar Return on Notional:** **2.43%** mean in Fear vs **5.80%** in Greed (`Rank-Biserial = -0.149`, FDR `p ≈ 0`), showing higher percentage efficiency per notional dollar in Greed despite lower total daily Fear volume.

### 5B. Trader Segmentation & Archetype Profiles (`06_trader_clusters.csv`, `06_cluster_summary.csv`)
- **Clustering Diagnostics:** Evaluated `KMeans` (`k = 2..7`) with `RobustScaler` on 11 complete (`32/32` non-null) behavioral, sizing, execution, risk-adjusted return, and contrarian features. Among `k >= 3`, **`k = 3`** achieved both the highest Silhouette Score (**`~0.237`**) and the lowest Davies-Bouldin Index (**`~1.10`**), with 2D PCA capturing **54.5%** of variance (`PC1 = 34.9%`, `PC2 = 19.6%`).
- **Three Validated Trader Archetypes (`32` accounts):**
  1. **`C0: Taker-Heavy` (`n = 24` accounts):** `182,124` trades, **\$522.17M** total volume, **+\$7.83M** net PnL after fees (`82.82%` mean win rate, `0.305` mean daily IR, **\$13,046.75** per USD 1M volume, `49.23%` mean taker, `48.39%` contrarian open share, `-\$56.82K` mean max daily drawdown).
  2. **`C1: Taker-Heavy Alpha Contrarian` (`n = 5` accounts):** Compact volume (**\$9.59M** across `4,442` trades) but generated **+\$1.20M** net PnL after fees — **\$121,353.91 mean PnL per USD 1M volume** (`94.04%` win rate, `0.384` daily IR, `64.94%` contrarian open share, `25.78%` spot share, **+18.24 pp** Fear-minus-Greed contrarian bias shift, and shallowest mean max drawdown at **-\$5.11K**).
  3. **`C2: High-Vol Taker-Heavy Low-Margin` (`n = 3` accounts: `0x4f93...f760`, `0x513b...4ff1`, `0xb899...2115`):** Accounted for **\$659.43M** (**55.4%** of platform notional volume) across `24,658` trades with **88.09%** taker execution and only **2.76%** spot, yet realized only **+\$1.03M** net PnL after fees — just **\$1,528.71 per USD 1M volume**, lowest daily IR (**0.241**), and deepest mean max daily drawdown (**-\$81.37K**).

### 5C. Regime Transitions, Contrarian vs Momentum Cohorts & Predictive Check (`07_signals_summary.csv`)
- **Regime-Transition Event Study (`t-3 .. t+5` calendar window):** Across `Shift into Fear (n = 31)` vs `Shift into Greed (n = 26)`, perp `Open-Long Share` climbs above **~72% at `t-1`** before `Fear` onset and dips to **~45% at `t+1`**, while calendar daily net PnL stays around **USD 17K–21K/day** over `t = 0..t+3`. Conversely, `Shift into Greed` sees volume peak near **~USD 4.2M at `t-1`** before daily net PnL compresses toward **~USD 0 at `t+2`**.
- **Contrarian vs Momentum Cohort Realized Performance (Non-Zero Closing Attribution):** Grouping all 32 accounts by contrarian open tendency reveals sharp monotonic risk-adjusted separation:
  - **Contrarian-Tilted (`>= 60%`, `n = 13`):** **+\$6,026,868.60** cumulative net PnL after fees, Max Cohort Drawdown **-\$51,642.28**.
  - **Balanced (`40–60%`, `n = 8`):** **+\$3,500,360.63** cumulative net PnL after fees, Max Cohort Drawdown **-\$170,721.32**.
  - **Momentum-Tilted (`< 40%`, `n = 11`):** **+\$523,871.99** cumulative net PnL after fees, Max Cohort Drawdown **-\$460,656.79** (`~8.9x` larger drawdown and `~11.5x` lower cumulative PnL than the Contrarian-Tilted cohort!).
  - Across Fear & Greed trading days, **Contrarian-Majority Days** also outperformed **Momentum-Majority Days** on mean daily net PnL in both Fear (`$43.47K` vs `$31.10K`, `n = 60` vs `39`) and Greed (`$24.26K` vs `$8.34K`, `n = 151` vs `156`). Account-level linear correlation between `contrarian_open_share_pct` and `daily_pnl_ir` across all 32 traders alone is weak (`Spearman r = +0.085, p = 0.642`), indicating that execution discipline & sizing interact strongly with contrarian bias.
- **Time-Aware Predictive Check (`5-fold TimeSeriesSplit`, `N = 352` strictly lagged daily rows):**
  - Linear `LogisticRegression` achieved modest above-chance out-of-sample discrimination (`CV ROC-AUC = 0.579` for `target_high_vol`; `CV ROC-AUC = 0.582` for `target_profitable`), whereas non-linear `RandomForestClassifier` hovered near random (`0.503` and `0.486`), demonstrating that strictly lagged daily behavioral/sentiment predictors carry genuine **regime/conditioning value** rather than standalone next-day alpha forecasting.
- **Visual Outputs Inspected:**
  - `outputs/07_stats_summary_table.png`, `outputs/07_stats_hypothesis_tests.png`, `outputs/07_stats_bootstrap_ci.png`, `outputs/08_clusters_k_selection.png`, `outputs/08_clusters_projection.png`, `outputs/08_clusters_profiles.png`, `outputs/08_clusters_regime_performance.png`, `outputs/09_signals_regime_transition_event_study.png`, `outputs/09_signals_contrarian_vs_momentum.png`, `outputs/09_signals_feature_importance.png`

---

## Phase 6: Insights Synthesis & Final Report (`ds_report.pdf`)

- **Status:** Generated & Visually Inspected (`ds_report.pdf`, **6 pages**, **3,905.3 KB** via `fpdf2`); **Page 1 minor layout & prose fix patched** in `notebook_2.ipynb` Section 5 (re-run Section 5 to refresh PDF).
- **Verified Summary Across Report (`ds_report.pdf` + `notebook_2.ipynb`):**
  - **Global Totals (`211,224` fills / `32` accounts / `246` coins / `476` active UTC days):** **\$1,191.19M** total notional volume, **+\$10.30M** gross closed PnL, **\$245.9K** total fees paid, **+\$10.05M** net PnL after fees, and **83.20%** global closing-trade win rate (`104,408` closing fills).
  - **Report Inspection Notes:**
    - Visual inspection of all 6 pages confirmed Pages 2–6 rendered cleanly with zero chart clipping or text overlap across Figures 1–9 and the Cluster Summary table.
    - Page 1 inspection caught two issues that have now been fixed in `notebook_2.ipynb` (`nb2-sec5-code`): (1) `13.57%` typo corrected dynamically to `{100.0 * n_date_shifted / tot_trades:.2f}%` (**25.24%** for `53,322` midnight-shifted trades), and (2) `Operational Context / Scope` column in the KPI scorecard changed from right-aligned (`R`) to left-aligned (`L`) with tightened column widths `[48, 56, 82]` so cell text stays cleanly inside borders.
- **Repository Finalization:**
  - Updated [`requirements.txt`](../requirements.txt) with all direct and key transitive runtime dependencies (`tzdata`, `pytz`, `python-dateutil`, `six`, `fonttools`, `defusedxml`, `joblib`, `threadpoolctl`, `patsy`, `contourpy`, `cycler`, `kiwisolver`, `packaging`, `pyparsing`, `ipython`, `ipywidgets`).
  - Updated [`README.md`](../README.md) with verified quantitative results, repository inventory, and local Jupyter setup instructions.
