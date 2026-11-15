# Traders Behavior Analysis — Execution Plan & Checklists

> **Project:** `Traders-Behavior-Analysis`  
> **Scope:** Analysing Trader Behavior with respect to Market Risk & Sentiment  
> **Execution Environment:** Local Jupyter Notebooks (`notebook_1.ipynb`, `notebook_2.ipynb`) — **no Google Colab dependencies**  
> **Last Updated:** 2026-11-15  

---

## 1. Repository Structure & Role of Each Directory

```text
Traders-Behavior-Analysis/
├── notebook_1.ipynb        # Phase 1–4: Data Ingestion, Cleaning, Alignment, EDA & Feature Engineering
├── notebook_2.ipynb        # Phase 5–6: Statistical Testing, Trader Segmentation, Signal Discovery & Report Visuals
├── csv_files/              # Raw, intermediate, and processed CSV datasets + summary metric tables
├── outputs/                # High-resolution charts, EDA figures, summary table snapshots, and progress images (*.png / *.jpg)
├── docs/
│   └── PLAN.md             # This step-by-step execution plan and progress checklists
├── ds_report.pdf           # Final compiled executive & technical insights report (generated in Phase 6)
├── requirements.txt        # Reproducible local Python / Jupyter environment dependencies
└── README.md               # Project overview, local setup instructions, and key findings summary
```

---

## 2. Master Progress Tracker

| Phase | Title | Primary Notebook / Deliverable | Status |
| :--- | :--- | :--- | :--- |
| **Phase 0** | Environment, Workspace & Conventions Setup | `requirements.txt`, `README.md`, `docs/PLAN.md` | `[x]` Completed |
| **Phase 1** | Data Acquisition & Initial Schema Inspection | `notebook_1.ipynb`, `csv_files/` | `[x]` Completed |
| **Phase 2** | Data Cleaning, Normalization & Temporal Alignment | `notebook_1.ipynb`, `csv_files/01_*.csv`, `outputs/01_*.png` | `[x]` Completed |
| **Phase 3** | Exploratory Data Analysis (EDA) & Visual Logging | `notebook_1.ipynb`, `outputs/02_*` – `05_*.png` | `[x]` Completed |
| **Phase 4** | Feature Engineering & Behavioral/Risk Metrics | `notebook_1.ipynb`, `csv_files/03_*.csv`, `csv_files/04_*.csv`, `outputs/06_*.png` | `[~]` Code Ready |
| **Phase 5** | Statistical Testing, Segmentation & Deep-Dive Modeling | `notebook_2.ipynb`, `csv_files/03_*.csv`, `outputs/07_*` – `09_*.png` | `[ ]` Not Started |
| **Phase 6** | Synthesis, Strategy Signals & Final Report Compilation | `notebook_2.ipynb`, `ds_report.pdf`, `README.md` | `[ ]` Not Started |

---

## 3. Core Engineering & Reproducibility Standards

### 3.1 Local Jupyter Workflow (Continuation Protocol)
- All notebook paths must resolve relative to the repository root using `pathlib.Path` (never Colab `/content/drive` mounts or hardcoded absolute user paths).
- Each notebook begins with a **Configuration & Helper Cell** defining:
  - `PROJECT_ROOT`, `CSV_DIR = PROJECT_ROOT / "csv_files"`, `OUTPUT_DIR = PROJECT_ROOT / "outputs"`.
  - Global visual theme (consistent palette, font sizes, grid styling, `dpi=300`, `bbox_inches="tight"`).
  - Helper functions:
    - `save_fig(fig, filename)` — saves `.png` (and optional `.jpg`) into `outputs/` and prints the saved path.
    - `save_table_image(df, filename, title)` — renders key pandas summary tables / KPI snapshots as crisp `.png` images in `outputs/` so tabular milestones are preserved visually alongside plots.
    - `export_csv(df, filename)` — writes processed DataFrames to `csv_files/` with row/column logging.
- Notebooks must run cleanly top-to-bottom via **Restart Kernel & Run All Cells** before marking a phase complete.
- Every checkpoint writes intermediate CSVs to `csv_files/` so `notebook_2.ipynb` (or a future session) can resume directly from processed artifacts without re-running raw preprocessing.

### 3.2 File Naming Conventions
- **CSVs in `csv_files/`:**
  - Raw inputs: `raw_<dataset_name>.csv` (never overwrite raw files in place).
  - Cleaned single datasets: `01_clean_<dataset_name>.csv`
  - Merged / aligned dataset: `02_merged_trader_sentiment.csv`
  - Engineered feature tables: `03_features_daily_regime.csv`, `04_features_trader_profiles.csv`
  - Statistical & modeling outputs: `05_stats_regime_tests.csv`, `06_trader_clusters.csv`
- **Images in `outputs/`:**
  - Prefix with a two-digit sequence number matching the analysis stage:
    - `01_audit_*.png` — missingness heatmaps, schema/coverage timelines, data quality cards
    - `02_eda_sentiment_*.png` — market sentiment / risk index distributions & regime timelines
    - `03_eda_traders_*.png` — PnL, volume, trade size, leverage, side/symbol distributions
    - `04_eda_regime_comparison_*.png` — trader behavior broken down across market risk/sentiment regimes
    - `05_eda_correlations_*.png` — correlation matrices, pairplots, and temporal lag plots
    - `06_features_*.png` — engineered risk/behavior metric distributions & summary KPI table images
    - `07_stats_*.png` — hypothesis test visualizations, confidence intervals, bootstrap distributions
    - `08_clusters_*.png` — trader segmentation (PCA/UMAP/cluster radar & profile comparisons)
    - `09_signals_*.png` — actionable strategy backtests, drawdown curves, regime-transition heatmaps

---

## 4. Detailed Step-by-Step Plan & Checklists

### Phase 0: Environment, Workspace & Conventions Setup
**Goal:** Establish a clean, reproducible local Python + Jupyter workspace before loading data.

- [x] Create standardized directory layout inside `Traders-Behavior-Analysis/`:
  - [x] `csv_files/`
  - [x] `outputs/`
  - [x] `docs/`
  - [x] `notebook_1.ipynb` (starter scaffold)
  - [x] `notebook_2.ipynb` (starter scaffold)
- [x] Create `requirements.txt` with core data science, visualization, statistics, and Jupyter packages.
- [x] Create `docs/PLAN.md` with step-by-step phases and checklists.
- [x] Create and activate a local Python virtual environment (`.venv/`) and install `requirements.txt`.
- [x] Register Jupyter kernel (`python -m ipykernel install --user --name traders-behavior`) and verify `notebook_1.ipynb` launches locally.
- [ ] Confirm `.gitignore` excludes `.venv/`, `.ipynb_checkpoints/`, and temporary cache files while keeping intentional outputs/docs tracked as desired.

---

### Phase 1: Data Acquisition & Initial Schema Inspection (`notebook_1.ipynb`)
**Goal:** Place raw datasets into `csv_files/` and perform a complete non-destructive structural audit.

- [x] Place raw trader dataset and market sentiment / market risk dataset into `csv_files/` (`fear_greed_index.csv`, `historical_data.csv`).
- [x] Document dataset provenance, collection timeframe, and file sizes in `notebook_1.ipynb` Markdown notes / `docs/FINDINGS.md`.
- [x] Load datasets into pandas and inspect:
  - [x] Shape (row count, column count) and memory footprint (`df.info(memory_usage="deep")`).
  - [x] Exact column names, raw dtypes, and sample head/tail rows.
  - [x] Unique identifier cardinality (number of distinct accounts/traders, symbols/assets, sentiment classes).
  - [x] Date/time range coverage, granularity (tick/trade-level vs daily), and timezone indicators.
- [x] **Visual & Tabular Progress Artifacts:**
  - [x] Export initial data dictionary & summary profile table to `csv_files/00_data_audit_summary.csv`.
  - [x] Save `outputs/01_audit_schema_summary.png` (rendered summary card of row counts, date ranges, and column types).
  - [x] Save `outputs/01_audit_missingness.png` (missing value count/percentage bar chart or matrix per dataset).

---

### Phase 2: Data Cleaning, Normalization & Temporal Alignment (`notebook_1.ipynb`)
**Goal:** Produce trustworthy, analysis-ready datasets with verified timestamp alignment between trader events and market risk/sentiment states.

- [x] **Column & Type Standardization:**
  - [x] Standardize column names to `snake_case` (strip whitespace, normalize casing).
  - [x] Cast numeric columns (`execution_price`, `size_tokens`, `size_usd`, `start_position`, `closed_pnl`, `fee`, `value`, etc.) safely with coercion checks; inspect any failed parses.
  - [x] Standardize categorical labels (`side`, `direction`, `crossed`, `classification`).
- [x] **Timestamp Parsing & Timezone Alignment:**
  - [x] Parse trader execution timestamps (`Timestamp IST` -> `UTC`) and market sentiment dates into timezone-aware UTC `datetime64`.
  - [x] Extract `trade_date_utc`, `trade_date_ist`, `hour_utc`, and `day_of_week_utc` columns.
  - [x] Check for overlap window between trader records and market sentiment dates; quantify IST-vs-UTC boundary shifts.
- [x] **Data Quality Remediation:**
  - [x] Identify and quantify exact duplicate rows and near-duplicate executions; preserve valid partial fills.
  - [x] Audit missing values (`0` missing cells) and verify notional consistency (`0` mismatches).
  - [x] Inspect extreme values / outliers (`> P99.9` size/PnL); keep raw values intact and add diagnostic flag columns.
- [x] **Dataset Merging & Integrity Verification:**
  - [x] Join trader records with daily (and lagged `t-1`) market sentiment/risk states on `trade_date_utc`.
  - [x] Assert post-merge row count preservation (`211,224 -> 211,224`).
  - [x] Verify zero unexpected `NaN`s in key join columns (`100.00%` matched).
- [x] **Checkpoint Exports:**
  - [x] Save cleaned datasets to `csv_files/01_clean_trader_data.csv` and `csv_files/01_clean_sentiment.csv`.
  - [x] Save merged dataset to `csv_files/02_merged_trader_sentiment.csv` and `csv_files/02_phase2_cleaning_summary.csv`.
  - [x] Save `outputs/01_audit_cleaning_funnel.png`, `outputs/01_audit_temporal_coverage.png`, and `outputs/01_audit_phase2_summary_table.png`.

---

### Phase 3: Exploratory Data Analysis (EDA) & Visual Logging (`notebook_1.ipynb`)
**Goal:** Systematically uncover distributions, anomalies, time trends, and behavioral shifts across market risk/sentiment regimes, saving every major finding as an image in `outputs/`.

#### 3A. Market Sentiment / Risk Regime Exploration
- [x] Compute summary statistics and class frequencies of market sentiment/risk regimes (counts, percentages, streak durations).
- [x] Plot full-period sentiment index & regime time series with trader-window comparison -> `outputs/02_eda_sentiment_timeline.png`.
- [x] Plot distribution of streak durations and next-day transition probability matrix -> `outputs/02_eda_sentiment_transitions.png`.

#### 3B. Trader Activity, Volume & Profitability Overview
- [x] Compute global trading KPIs: total trades, active traders, total notional volume, net/gross closed PnL, win rate, median vs mean trade size, maker/taker & spot/perp mix.
- [x] Render KPI summary table as image -> `outputs/03_eda_global_kpis_table.png`.
- [x] Plot PnL distribution & cumulative PnL curve -> `outputs/03_eda_pnl_distribution.png`.
- [x] Plot log trade size & execution direction breakdown -> `outputs/03_eda_volume_distribution.png`.
- [x] Plot directional side (`BUY`/`SELL`), maker/taker role & perp-vs-spot breakdown -> `outputs/03_eda_execution_and_side.png`.
- [x] Plot top traded symbols/assets by volume and net PnL -> `outputs/03_eda_symbol_breakdown.png`.
- [x] Plot trader concentration (Lorenz curve with Gini + all 32 accounts PnL ranking) -> `outputs/03_eda_trader_concentration.png`.

#### 3C. Behavior Across Market Risk & Sentiment Regimes
- [x] Compare daily trade count, active trader count, and notional volume across sentiment regimes -> `outputs/04_eda_activity_by_regime.png`.
- [x] Compare profitability metrics (mean closed PnL, win rate, profit factor, daily net PnL) across sentiment regimes -> `outputs/04_eda_pnl_by_regime.png`.
- [x] Compare sizing, taker aggression, and Open-Long bias across sentiment regimes -> `outputs/04_eda_risk_and_bias_by_regime.png`.
- [x] Plot Day-of-Week × UTC Hour trading intensity heatmaps conditioned on Fear vs Greed -> `outputs/04_eda_temporal_heatmap_by_regime.png`.

#### 3D. Correlation, Lead-Lag & Multi-Dimensional Relationships
- [x] Compute Spearman & Pearson correlation matrices across daily aggregated behavioral metrics and sentiment score -> `outputs/05_eda_correlation_heatmap.png`.
- [x] Analyze exact calendar-day lead-lag relationships (`k = -5..+5`) -> `outputs/05_eda_lead_lag_analysis.png`.
- [x] Capture written EDA observations & hypotheses in `docs/FINDINGS.md`.

---

### Phase 4: Feature Engineering & Behavioral/Risk Metrics (`notebook_1.ipynb`)
**Goal:** Transform raw trade-level records into rich daily-regime features and account-level behavioral profiles for rigorous modeling in `notebook_2.ipynb`.

- [~] **Trade-Level Enriched Features:**
  - [~] `net_pnl_after_fee`, `is_win`, `is_loss`
  - [~] `return_on_notional_pct = 100 * closed_pnl / size_usd`
  - [~] `fee_bps` and `position_turnover_ratio`
  - [~] `is_contrarian_open` & `is_momentum_open`
  - [~] `regime_streak_days`, `regime_transition_flag`, `sentiment_7d_ma`, `sentiment_7d_vol`, `sentiment_value_lag3`
- [~] **Daily Aggregated Behavioral Dataset (`03_features_daily_regime.csv`):**
  - [~] Daily trade count, active accounts, unique coins, total volume, median/mean/P90 trade size, taker %, spot %, win rate, profit factor, net PnL, PnL dispersion (`pnl_std_usd`), max single loss.
  - [~] Daily `open_long_share_pct`, `contrarian_open_pct`, and backward-looking 7-day rolling totals (`roll7d_trades`, `roll7d_volume_usd`, `roll7d_net_pnl`).
- [~] **Trader-Level Behavioral Profile Dataset (`04_features_trader_profiles.csv`):**
  - [~] Activity metrics: `total_trades`, `active_days`, `trades_per_active_day`, `span_calendar_days`, `unique_coins`, `top_coin`, `top_coin_vol_share_pct`.
  - [~] Performance & risk metrics: `net_pnl_usd`, `win_rate_pct`, `profit_factor`, `pnl_per_1m_volume`, `daily_pnl_ir`, `max_daily_drawdown_usd`, `profitable_days_pct`.
  - [~] Execution & regime-conditional metrics: `taker_pct`, `spot_pct`, `avg_fee_bps`, `size_cv`, Fear-vs-Greed volume/PnL/win-rate splits, `contrarian_open_share_pct`, `contrarian_bias_shift`.
- [ ] **Feature Validation & Visual Exports (run Phase 4 cells):**
  - [ ] Verify zero infinite values and zero look-ahead leakage in engineered features.
  - [ ] Export `csv_files/03_features_daily_regime.csv` and `csv_files/04_features_trader_profiles.csv`.
  - [ ] Save `outputs/06_features_distributions.png`, `outputs/06_features_summary_table.png`, and `outputs/06_features_fear_vs_greed_trader_scatter.png`.

---

### Phase 5: Statistical Testing, Trader Segmentation & Deep-Dive Modeling (`notebook_2.ipynb`)
**Goal:** Rigorously validate EDA hypotheses with statistical tests, segment traders into behavioral archetypes, and uncover predictive/actionable risk signals.

- [ ] **Load Processed Artifacts:**
  - [ ] Load `02_merged_trader_sentiment.csv`, `03_features_daily_regime.csv`, and `04_features_trader_profiles.csv` from `csv_files/`.
  - [ ] Run schema & row-count assertions at top of `notebook_2.ipynb`.
- [ ] **Hypothesis Testing Across Regimes:**
  - [ ] Test whether trader PnL, win rate, notional volume, leverage, and long/short bias differ significantly across market sentiment/risk regimes:
    - [ ] Non-parametric tests (Mann-Whitney U / Kruskal-Wallis H) for skewed distributions (PnL, volume, leverage).
    - [ ] Proportion / contingency tests (Chi-Square / Fisher's Exact) for win rates and directional bias (`Long` vs `Short`).
    - [ ] Effect sizes (Rank-Biserial Correlation, Cliff's Delta, Cramér's V) and multiple-testing correction (Benjamini-Hochberg FDR).
    - [ ] Bootstrap 95% confidence intervals for regime-level mean/median differences.
  - [ ] Export test results table to `csv_files/05_stats_regime_tests.csv`.
  - [ ] Save `outputs/07_stats_hypothesis_tests.png` and `outputs/07_stats_bootstrap_ci.png`.
- [ ] **Trader Segmentation & Archetype Discovery:**
  - [ ] Filter or weight accounts by minimum activity threshold (avoid noisy 1-trade accounts skewing clustering; analyze low-activity cohort separately).
  - [ ] Scale/transform skewed trader features (log1p + RobustScaler / StandardScaler).
  - [ ] Determine optimal cluster count $k$ via Elbow, Silhouette Score, and Davies-Bouldin Index -> `outputs/08_clusters_k_selection.png`.
  - [ ] Fit clustering (K-Means / GMM / Hierarchical) and assign interpretable behavioral labels (e.g., *Disciplined Contrarians*, *High-Leverage Momentum Chasers*, *Institutional Whales*, *Over-Traded Loss Leaders*).
  - [ ] Visualize clusters via 2D PCA/UMAP scatter and normalized radar/heatmap profiles -> `outputs/08_clusters_projection.png`, `outputs/08_clusters_profiles.png`.
  - [ ] Analyze how each trader archetype performs under each market sentiment/risk regime -> `outputs/08_clusters_regime_performance.png`.
  - [ ] Export clustered profiles to `csv_files/06_trader_clusters.csv`.
- [ ] **Regime-Transition, Drawdown & Signal Analysis:**
  - [ ] Event study around sharp sentiment shifts (e.g., sudden transitions into Extreme Fear / Extreme Greed): track cumulative PnL, leverage spikes, and volume surges at $t-3 \dots t+5$ -> `outputs/09_signals_regime_transition_event_study.png`.
  - [ ] Compare **Contrarian vs Momentum** strategies across regimes (equity curves, max drawdown, risk-adjusted return) -> `outputs/09_signals_contrarian_vs_momentum.png`.
  - [ ] Optional predictive check: evaluate simple interpretable models (Logistic Regression / Random Forest / Gradient Boosting with time-aware CV) to predict profitable trade days or tail-loss risk from behavioral + sentiment features, inspecting feature importances (SHAP / permutation) -> `outputs/09_signals_feature_importance.png`.

---

### Phase 6: Insights Synthesis, Recommendations & Final Report (`notebook_2.ipynb` & `ds_report.pdf`)
**Goal:** Distill analytical findings into clear, evidence-backed insights, compile `ds_report.pdf`, and ensure the entire repository is publication-ready.

- [ ] **Synthesize Key Findings & Trading/Risk Signals:**
  - [ ] Summarize top 5–7 quantitative findings with exact effect sizes, confidence intervals, and supporting chart references.
  - [ ] Formulate concrete risk-management & strategy recommendations (e.g., leverage caps during high-greed/high-volatility regimes, contrarian sizing rules, regime-aware position filters).
  - [ ] Explicitly document data limitations, survivorship/selection biases, sample coverage gaps, and confounding factors.
- [ ] **Generate `ds_report.pdf`:**
  - [ ] Build a clean, reproducible report script or notebook section (e.g., using `fpdf2` or `matplotlib`/`weasyprint`) that embeds curated `outputs/*.png` charts, KPI summary tables, methodology, insights, and recommendations into `ds_report.pdf`.
  - [ ] Visually inspect every page of `ds_report.pdf` for layout, legibility, chart clipping, and accurate numbers.
- [ ] **Final Repository QA & Documentation:**
  - [ ] Re-run `notebook_1.ipynb` end-to-end (`Restart & Run All`) and confirm zero errors and clean outputs.
  - [ ] Re-run `notebook_2.ipynb` end-to-end (`Restart & Run All`) and confirm zero errors and clean outputs.
  - [ ] Update `README.md` with final project summary, folder inventory, setup/run instructions, and embedded preview of key charts.
  - [ ] Update all checkboxes in `docs/PLAN.md` to reflect final completion status.

---

## 5. Continuation Notes (For Picking Up Later)
1. **Current State:** Initial repository structure, local Jupyter starter notebooks (`notebook_1.ipynb`, `notebook_2.ipynb`), `requirements.txt`, and this plan (`docs/PLAN.md`) are ready.
2. **Immediate Next Step:** Place the raw CSV files into `csv_files/`, set up the local Python virtual environment (`pip install -r requirements.txt`), and start executing **Phase 1** in `notebook_1.ipynb`.
3. **Progress Tracking Rule:** Whenever a milestone figure or summary table is generated, save it immediately to `outputs/` using `save_fig(...)` or `save_table_image(...)` and tick the corresponding checkbox in this file.
