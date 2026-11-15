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

- **Status:** Code Ready in `notebook_1.ipynb` (awaiting user execution in Jupyter)
- **Planned Sections:**
  - **3A. Market Sentiment / Risk Regimes:** Full vs active-window regime distributions, transition matrix, streak durations (`outputs/02_eda_sentiment_*.png`).
  - **3B. Trader Activity, Volume, Concentration & Profitability:** Global KPIs table image, PnL & volume distributions, execution/side/spot-vs-perp mix, top coins, Lorenz concentration & account PnL ranking (`outputs/03_eda_*.png`).
  - **3C. Behavior Across Market Sentiment Regimes:** Daily-normalized activity, profitability/win-rate/profit-factor, directional bias & taker aggression, and Day×Hour intensity heatmaps (`outputs/04_eda_*.png`).
  - **3D. Correlation & Lead-Lag Analysis:** Daily Spearman/Pearson correlation heatmaps and lag `-5..+5` cross-correlations (`outputs/05_eda_*.png`).

---

## Phase 4: Feature Engineering & Behavioral/Risk Metrics

- **Status:** Not Started

---

## Phase 5: Statistical Testing, Trader Segmentation & Signal Discovery

- **Status:** Not Started

---

## Phase 6: Insights Synthesis & Final Report (`ds_report.pdf`)

- **Status:** Not Started
