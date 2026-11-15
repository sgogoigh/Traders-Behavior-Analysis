# Traders-Behavior-Analysis
Analysing Trader Behavior with respect to market risk

---

## Project Structure

```text
Traders-Behavior-Analysis/
├── notebook_1.ipynb        # Data ingestion, cleaning, alignment, EDA & feature engineering
├── notebook_2.ipynb        # Statistical testing, trader segmentation, signal discovery & reporting
├── csv_files/              # Raw, intermediate, and processed CSV datasets
├── outputs/                # Visual outputs, EDA charts, and summary table snapshots (*.png / *.jpg)
├── docs/
│   └── PLAN.md             # Comprehensive step-by-step execution plan and checklists
├── ds_report.pdf           # Final summarized insights and explanations (generated in Phase 6)
├── requirements.txt        # Local Python & Jupyter dependencies
└── README.md               # Project overview and local setup instructions
```

---

## Local Jupyter Setup

This project runs locally in **JupyterLab / Jupyter Notebook** (no cloud/Colab runtime required).

1. **Create and activate a virtual environment** (from `Traders-Behavior-Analysis/`):
   ```bash
   python -m venv .venv
   # Windows PowerShell:
   .venv\Scripts\Activate.ps1
   # macOS / Linux:
   source .venv/bin/activate
   ```
2. **Install dependencies and register the kernel**:
   ```bash
   pip install -r requirements.txt
   python -m ipykernel install --user --name traders-behavior --display-name "Python (Traders-Behavior)"
   ```
3. **Launch Jupyter**:
   ```bash
   jupyter lab
   ```
4. **Follow the roadmap**:
   See [`docs/PLAN.md`](docs/PLAN.md) for the complete phase-by-phase workflow, naming conventions, visual progress logging rules, and interactive checklists.
