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
| **Phase 0** | Environment, Workspace & Conventions Setup | `requirements.txt`, `README.md`, `docs/PLAN.md` | `[x]` Initialized |
| **Phase 1** | Data Acquisition & Initial Schema Inspection | `notebook_1.ipynb`, `csv_files/` | `[ ]` Pending Data |
| **Phase 2** | Data Cleaning, Normalization & Temporal Alignment | `notebook_1.ipynb`, `csv_files/01_*.csv`, `outputs/01_*.png` | `[ ]` Not Started |
| **Phase 3** | Exploratory Data Analysis (EDA) & Visual Logging | `notebook_1.ipynb`, `outputs/02_*` – `05_*.png` | `[ ]` Not Started |
| **Phase 4** | Feature Engineering & Behavioral/Risk Metrics | `notebook_1.ipynb`, `csv_files/02_*.csv`, `outputs/06_*.png` | `[ ]` Not Started |
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
- [ ] Create and activate a local Python virtual environment (`.venv/`) and install `requirements.txt`.
- [ ] Register Jupyter kernel (`python -m ipykernel install --user --name traders-behavior`) and verify `notebook_1.ipynb` launches locally.
- [ ] Confirm `.gitignore` excludes `.venv/`, `.ipynb_checkpoints/`, and temporary cache files while keeping intentional outputs/docs tracked as desired.

---

### Phase 1: Data Acquisition & Initial Schema Inspection (`notebook_1.ipynb`)
**Goal:** Place raw datasets into `csv_files/` and perform a complete non-destructive structural audit.

- [ ] Place raw trader dataset and market sentiment / market risk dataset into `csv_files/` (e.g., `raw_trader_data.csv`, `raw_market_sentiment.csv`).
- [ ] Document dataset provenance, collection timeframe, and file sizes in `notebook_1.ipynb` Markdown notes.
- [ ] Load datasets into pandas and inspect:
  - [ ] Shape (row count, column count) and memory footprint (`df.info(memory_usage="deep")`).
  - [ ] Exact column names, raw dtypes, and sample head/tail rows.
  - [ ] Unique identifier cardinality (number of distinct accounts/traders, symbols/assets, sentiment classes).
  - [ ] Date/time range coverage, granularity (tick/trade-level vs daily), and timezone indicators.
- [ ] **Visual & Tabular Progress Artifacts:**
  - [ ] Export initial data dictionary & summary profile table to `csv_files/00_data_audit_summary.csv`.
  - [ ] Save `outputs/01_audit_schema_summary.png` (rendered summary card of row counts, date ranges, and column types).
  - [ ] Save `outputs/01_audit_missingness.png` (missing value count/percentage bar chart or matrix per dataset).

---

### Phase 2: Data Cleaning, Normalization & Temporal Alignment (`notebook_1.ipynb`)
**Goal:** Produce trustworthy, analysis-ready datasets with verified timestamp alignment between trader events and market risk/sentiment states.

- [ ] **Column & Type Standardization:**
  - [ ] Standardize column names to `snake_case` (strip whitespace, normalize casing).
  - [ ] Cast numeric columns (`price`, `size`, `pnl`, `leverage`, `sentiment_value`, etc.) safely with coercion checks; inspect any failed parses.
  - [ ] Standardize categorical labels (e.g., trade side `BUY`/`SELL` or `Long`/`Short`, sentiment regime labels, event types).
- [ ] **Timestamp Parsing & Timezone Alignment:**
  - [ ] Parse trader execution timestamps and market sentiment dates into timezone-aware UTC `datetime64`.
  - [ ] Extract `trade_date` (UTC date), `hour_of_day`, `day_of_week`, and `week` columns.
  - [ ] Check for overlap window between trader records and market sentiment dates; quantify and log any out-of-range dates.
- [ ] **Data Quality Remediation:**
  - [ ] Identify and quantify exact duplicate rows and near-duplicate executions; document deduplication rules.
  - [ ] Audit missing values (drop vs impute vs flag) and justify every rule in Markdown.
  - [ ] Inspect extreme values / outliers (zero or negative prices/sizes, extreme leverage, huge PnL spikes); keep raw values intact and add outlier-flag columns rather than silently deleting legitimate whale trades.
- [ ] **Dataset Merging & Integrity Verification:**
  - [ ] Join trader records with daily (and lagged `t-1`, `t+1`) market sentiment/risk states on `trade_date`.
  - [ ] Assert post-merge row count preservation (no unintended many-to-many row explosion).
  - [ ] Verify zero unexpected `NaN`s in key join columns.
- [ ] **Checkpoint Exports:**
  - [ ] Save cleaned datasets to `csv_files/01_clean_trader_data.csv` and `csv_files/01_clean_sentiment.csv`.
  - [ ] Save merged dataset to `csv_files/02_merged_trader_sentiment.csv`.
  - [ ] Save `outputs/01_audit_cleaning_funnel.png` (before/after row retention & cleaning summary chart).
  - [ ] Save `outputs/01_audit_temporal_coverage.png` (daily trade volume vs sentiment coverage timeline).

---

### Phase 3: Exploratory Data Analysis (EDA) & Visual Logging (`notebook_1.ipynb`)
**Goal:** Systematically uncover distributions, anomalies, time trends, and behavioral shifts across market risk/sentiment regimes, saving every major finding as an image in `outputs/`.

#### 3A. Market Sentiment / Risk Regime Exploration
- [ ] Compute summary statistics and class frequencies of market sentiment/risk regimes (counts, percentages, streak durations).
- [ ] Plot full-period sentiment index & regime time series with regime transition markers -> `outputs/02_eda_sentiment_timeline.png`.
- [ ] Plot distribution of sentiment scores and transition matrix (probability of moving from one regime to another) -> `outputs/02_eda_sentiment_transitions.png`.

#### 3B. Trader Activity, Volume & Profitability Overview
- [ ] Compute global trading KPIs: total trades, active traders, total notional volume, net/gross closed PnL, win rate, median vs mean trade size, leverage distribution.
- [ ] Render KPI summary table as image -> `outputs/03_eda_global_kpis_table.png`.
- [ ] Plot PnL distribution (linear + symlog/clipped views to handle heavy tails) -> `outputs/03_eda_pnl_distribution.png`.
- [ ] Plot trade size & notional volume distributions (log-scale histograms / ECDF) -> `outputs/03_eda_volume_distribution.png`.
- [ ] Plot leverage & directional side (`Long` vs `Short`) breakdown -> `outputs/03_eda_leverage_and_side.png`.
- [ ] Plot top traded symbols/assets by trade count, volume, and net PnL -> `outputs/03_eda_symbol_breakdown.png`.
- [ ] Plot trader concentration (Lorenz curve / Pareto chart: share of volume & PnL generated by top 1%, 5%, 10% of accounts) -> `outputs/03_eda_trader_concentration.png`.

#### 3C. Behavior Across Market Risk & Sentiment Regimes
- [ ] Compare daily trade count, active trader count, and notional volume across sentiment regimes -> `outputs/04_eda_activity_by_regime.png`.
- [ ] Compare profitability metrics (mean/median closed PnL, win rate, profit factor, tail losses) across sentiment regimes -> `outputs/04_eda_pnl_by_regime.png`.
- [ ] Compare risk-taking behavior (leverage distribution, position size, long/short ratio, liquidation/loss rate) across sentiment regimes -> `outputs/04_eda_risk_by_regime.png`.
- [ ] Plot hourly / day-of-week trading intensity heatmaps conditioned on sentiment regime -> `outputs/04_eda_temporal_heatmap_by_regime.png`.

#### 3D. Correlation, Lead-Lag & Multi-Dimensional Relationships
- [ ] Compute Spearman & Pearson correlation matrices across daily aggregated behavioral metrics and sentiment score -> `outputs/05_eda_correlation_heatmap.png`.
- [ ] Analyze lead-lag relationships: does today's sentiment (`t`) correlate more strongly with today's behavior or next-day (`t+1`, `t+3`) trader positioning/PnL? -> `outputs/05_eda_lead_lag_analysis.png`.
- [ ] Capture written EDA observations & hypotheses directly in `notebook_1.ipynb` markdown cells after each figure group.

---

### Phase 4: Feature Engineering & Behavioral/Risk Metrics (`notebook_1.ipynb`)
**Goal:** Transform raw trade-level records into rich daily-regime features and account-level behavioral profiles for rigorous modeling in `notebook_2.ipynb`.

- [ ] **Trade-Level Enriched Features:**
  - [ ] `notional_usd = execution_price * size`
  - [ ] `is_win`, `is_loss`, `is_breakeven` flags based on closed PnL
  - [ ] `return_on_notional = closed_pnl / notional_usd` (with zero-division guard)
  - [ ] `risk_adjusted_pnl = closed_pnl / (notional_usd / leverage)` (margin-based return proxy, where leverage > 0)
  - [ ] `regime_change_flag` & `regime_streak_days` (captures market regime transitions vs steady states)
  - [ ] `lagged_sentiment_1d`, `lagged_sentiment_3d`, `sentiment_delta_1d`
- [ ] **Daily Aggregated Behavioral Dataset (`03_features_daily_regime.csv`):**
  - [ ] Daily trade count, unique active traders, total notional volume, net PnL, median PnL, win rate, profit factor.
  - [ ] Daily average & 90th-percentile leverage, long/short volume ratio, PnL dispersion (std / IQR).
  - [ ] Daily rolling metrics (7-day rolling volume, 7-day rolling PnL, 7-day sentiment volatility).
- [ ] **Trader-Level Behavioral Profile Dataset (`04_features_trader_profiles.csv`):**
  - [ ] Activity metrics: total trades, active days, trades per active day, preferred symbols, average holding/session cadence.
  - [ ] Performance metrics: cumulative PnL, mean/median PnL per trade, win rate, profit factor, max single-trade drawdown, Sharpe-like daily PnL consistency ratio.
  - [ ] Risk & alignment metrics: mean/max leverage, position size consistency (CV), overall long bias, Fear-vs-Greed PnL split, Fear-vs-Greed leverage shift, **Contrarian Alignment Score** (tendency to buy/long during Fear and sell/short during Greed).
- [ ] **Feature Validation & Visual Exports:**
  - [ ] Verify no infinite values, NaN leakage, or look-ahead leakage in engineered features.
  - [ ] Export `csv_files/03_features_daily_regime.csv` and `csv_files/04_features_trader_profiles.csv`.
  - [ ] Save `outputs/06_features_distributions.png` (key engineered behavioral & risk feature distributions).
  - [ ] Save `outputs/06_features_summary_table.png` (feature dictionary & descriptive stats snapshot).

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
