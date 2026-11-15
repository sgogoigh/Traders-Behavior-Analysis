# Traders Behavior Analysis — Findings Log

> **Project:** `Traders-Behavior-Analysis`  
> **Companion Plan:** [`docs/PLAN.md`](PLAN.md)  
> **Note:** Findings are appended sequentially by phase and section strictly from inspected files, executed notebook outputs, exported CSVs, and generated visuals.

---

## Phase 0: Environment & Repository Initialization

- **Status:** Completed
- **Summary:**
  - Initialized repository directory structure: `csv_files/`, `outputs/`, `docs/`.
  - Created local Jupyter starter notebooks [`notebook_1.ipynb`](../notebook_1.ipynb) and [`notebook_2.ipynb`](../notebook_2.ipynb) with relative path configuration (`pathlib.Path`) and helpers (`save_fig`, `save_table_image`, `export_csv`).
  - Added [`requirements.txt`](../requirements.txt), [`docs/PLAN.md`](PLAN.md), and updated [`README.md`](../README.md).

---

## Phase 1: Data Acquisition & Initial Schema Inspection

- **Status:** Code Ready in `notebook_1.ipynb` (initial file inspection completed; awaiting notebook execution for full column audit & visuals)
- **1.1 Raw Dataset Inventory (Observed from `dataset/`):**
  - `dataset/fear_greed_index.csv` — **2,645 lines** (1 header + **2,644 rows**), 4 columns: `timestamp`, `value`, `classification`, `date`.
    - Sample span observed directly in file: `2018-02-01` (`timestamp=1517463000`, `value=30`, `classification=Fear`) through `2025-05-02` (`timestamp=1746163800`, `value=67`, `classification=Greed`).
  - `dataset/historical_data.csv` — **211,225 lines** (1 header + **211,224 rows**), 16 columns: `Account`, `Coin`, `Execution Price`, `Size Tokens`, `Size USD`, `Side`, `Timestamp IST`, `Start Position`, `Direction`, `Closed PnL`, `Transaction Hash`, `Order ID`, `Crossed`, `Fee`, `Trade ID`, `Timestamp`.
- **1.2 Early Data-Quality Observations (Direct File Inspection):**
  - **Lossy scientific notation in `Timestamp` and `Trade ID`:** In `historical_data.csv`, `Timestamp` (`1.73E+12`, `1.75E+12`) and `Trade ID` (`8.95E+14`, `1.04E+15`) were exported in 3-significant-digit scientific notation, losing precision. `Timestamp IST` (`%d-%m-%Y %H:%M`, e.g., `02-12-2024 22:50`) retains minute-level precision and must be used as the authoritative timestamp.
  - **Multiple partial fills per order/tx:** Distinct execution rows share the same `Transaction Hash` and `Order ID` with different `Execution Price`, `Size Tokens`, and `Start Position`, so deduplication must only target exact full-row duplicates.
  - **Asset & Schema Scope:** `Coin` contains both perpetual tickers (e.g., `AAVE`, `FARTCOIN`) and spot index identifiers (e.g., `@107`). Notably, there is **no `Leverage` column** in `historical_data.csv`, so risk/exposure metrics in later phases will rely on `Size USD`, `Start Position`, `Fee`, `Crossed` (maker/taker), `Direction`, and `Closed PnL`.
- **1.3 Pending Notebook Execution Outputs:**
  - `csv_files/00_data_audit_summary.csv`
  - `outputs/01_audit_schema_summary.png`, `outputs/01_audit_missingness.png`

---

## Phase 2: Data Cleaning, Normalization & Temporal Alignment

- **Status:** Code Ready in `notebook_1.ipynb` (awaiting user execution in Jupyter)
- **Implemented Pipeline Sections:**
  - **2.1 Sentiment Cleaning (`01_clean_sentiment.csv`):** Normalizes headers to `snake_case`, parses `date` and Unix `timestamp`, checks calendar continuity for missing days, builds 3-bucket `regime_coarse` (`Fear` / `Neutral` / `Greed`), ordered encoding `regime_score_ord`, and 1-day lagged features (`sentiment_value_lag1`, `sentiment_class_lag1`, `sentiment_delta_1d`).
  - **2.2 Trader Cleaning (`01_clean_trader_data.csv`):** Converts `Timestamp IST` (`Asia/Kolkata`) to `UTC` (`timestamp_utc`, `trade_date_utc`, `hour_utc`, `day_of_week_utc`), casts numerics & `crossed` boolean safely, removes exact full-row duplicates only, and adds non-destructive flags (`is_spot_index`, `is_maker`, `is_closing_trade`, `is_negative_fee`, `notional_mismatch_flag`, `extreme_size_flag`, `extreme_pnl_flag`, `date_shifted_ist_to_utc`).
  - **2.3 Temporal Merge (`02_merged_trader_sentiment.csv`):** `many_to_one` left-join on `trade_date_utc` with row-preservation assertion and IST-vs-UTC overlap diagnostics.
  - **2.4 Pending Audit Artifacts (generated on run):**
    - `csv_files/02_phase2_cleaning_summary.csv`
    - `outputs/01_audit_phase2_summary_table.png`, `outputs/01_audit_cleaning_funnel.png`, `outputs/01_audit_temporal_coverage.png`
- **Observed Results:** *(To be appended once `notebook_1.ipynb` Phases 1–2 are executed.)*

---

## Phase 3: Exploratory Data Analysis (EDA)

- **Status:** Not Started
- **Planned Sections:** `3A` Sentiment Regimes | `3B` Trader Activity & Profitability | `3C` Behavior Across Regimes | `3D` Correlations & Lead-Lag

---

## Phase 4: Feature Engineering & Behavioral/Risk Metrics

- **Status:** Not Started

---

## Phase 5: Statistical Testing, Trader Segmentation & Signal Discovery

- **Status:** Not Started

---

## Phase 6: Insights Synthesis & Final Report (`ds_report.pdf`)

- **Status:** Not Started
