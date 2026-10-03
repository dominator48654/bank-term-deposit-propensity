# Customer Propensity & Campaign Valuation — Bank Term-Deposit Telemarketing

> **A production-grade analytics notebook** that scores 41,188 outbound-call customers for term-deposit propensity, values each call in rupee terms, and recommends channel / contact-frequency policy — built as a portfolio piece for Axis Bank BIU (Business Intelligence Unit) Analytics.

---

## Business Problem

A bank's outbound call centre sells term deposits. Every call costs money and most calls don't convert. This project answers four questions:

| # | Question | Notebook Section |
|---|----------|------------------|
| 1 | **Whom should we call?** — propensity score that ranks customers | §5 – §7, §9, §10 |
| 2 | **Is the campaign worth running, and for whom?** — unit economics | §11 |
| 3 | **Through which channel, and how many times?** — channel & contact policy | §12 |
| 4 | **Why did performance change?** — root-cause analysis of conversion drift | §4, §8 |

## Key Design Decisions

Three choices separate this from a typical tutorial:

1. **`duration` (call length) is excluded.** It is only known *after* the call ends; a model that uses it cannot decide *whom* to call.
2. **Validation is out-of-time, not a random split.** The data is chronological and conversion swings from ~5 % to ~52 %. A random split lets the model learn from the future.
3. **Macro indicators (Euribor, employment, CPI) are kept out of the ranking model.** They are identical for everyone called on a given day, so they tell you *when* conversion will be high, not *whom* to call.

## Results at a Glance

| Metric | Value |
|--------|-------|
| Holdout AUC (2010 out-of-time) | **0.79** |
| Top-decile lift | **1.5×** |
| Walk-forward monthly AUC (Nov '08 – Nov '10) | 0.52 – 0.76 (mean ≈ 0.62) |
| Bottom-3-decile skip rule (low-conversion regime) | Cuts ~30 % of calls, retains ~75 % of subscribers |

## Repository Structure

```
.
├── bank_term_deposit_propensity.ipynb   # Main analysis notebook (end-to-end)
├── data/
│   └── bank-additional-full.csv         # UCI Bank Marketing dataset (41,188 rows)
├── outputs/                             # Pre-computed CSV tables (for dashboards)
│   ├── monthly_kpis_and_model_auc.csv   # Monthly volume, conversion, AUC
│   ├── holdout_gains_table.csv          # Decile gains table on 2010 holdout
│   ├── holdout_2010_scored.csv          # Row-level scores on holdout
│   ├── decile_pnl_by_regime.csv         # Profit by decile × regime
│   ├── pnl_by_year.csv                  # Year-level P&L
│   ├── model_selection_backtest.csv     # Walk-forward model comparison
│   ├── driver_odds_ratios.csv           # Logistic-regression odds ratios
│   ├── segment_index_age.csv            # Month-adjusted conversion by age band
│   ├── segment_index_job.csv            # Month-adjusted conversion by job
│   ├── skip_rule_low_conversion.csv     # Impact of dropping low deciles
│   ├── validation_trap.csv              # Random-split vs out-of-time comparison
│   ├── f1_by_window.csv                 # F1 across calibration windows
│   ├── f1_holdout_rules.csv             # F1 with various decision rules
│   ├── f1_precision_improvement_attempt.csv
│   └── precision_operating_points.csv
├── requirements.txt
├── LICENSE
└── README.md
```

## Notebook Sections

| # | Section | What it covers |
|---|---------|----------------|
| 1 | Setup & data loading | Imports, dataset extraction |
| 2 | Timeline rebuild & data audit | Leakage flags, sentinel values, rare categories |
| 3 | Exploratory analysis | Volume/conversion trends, pockets of opportunity |
| 4 | Root-cause analysis | Why conversion jumped from ~5 % to ~52 % |
| 5 | Validation trap | Random split vs. out-of-time — why it matters |
| 6 | Model selection | Logistic regression vs. gradient boosting (walk-forward) |
| 7 | Final holdout evaluation | 2010 out-of-time test, gains table, feature-set audit |
| 8 | Driver analysis | Odds ratios with month fixed effects |
| 9 | Calibration under drift | Recalibration, F1/precision operating points |
| 10 | Deployment simulation | Monthly walk-forward scoring & monitoring KPIs |
| 11 | Valuation | Per-call P&L, decile-level economics, sensitivity |
| 12 | Channel & contact-frequency | Mobile vs. landline, marginal-call break-even |
| 13 | Summary & recommendations | Five actionable recommendations |
| 14 | Export | CSV tables for Excel / Power BI dashboards |

## Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/<your-username>/bank-term-deposit-propensity.git
cd bank-term-deposit-propensity

# 2. Create a virtual environment (optional but recommended)
python -m venv .venv
source .venv/bin/activate        # Linux / macOS
# .venv\Scripts\activate         # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Launch the notebook
jupyter notebook bank_term_deposit_propensity.ipynb
```

> The notebook downloads and extracts the dataset from the bundled zip automatically on first run. If you prefer, place `bank-additional-full.csv` in `data/` manually.

## Data Source

[UCI Machine Learning Repository — Bank Marketing Data Set](https://archive.ics.uci.edu/ml/datasets/Bank+Marketing)

> S. Moro, P. Cortez and P. Rita. *A Data-Driven Approach to Predict the Success of Bank Telemarketing.* Decision Support Systems, 62, 2014, pp. 22–31.

## Tech Stack

| Layer | Tools |
|-------|-------|
| Language | Python 3.10+ |
| Data | pandas, NumPy |
| Modelling | scikit-learn (Logistic Regression, HistGradientBoosting), statsmodels |
| Visualisation | Matplotlib |
| Statistics | SciPy |

## Recommendations (from §13)

1. **Score before every wave.** Call in score order; drop low deciles when the base rate is below break-even.
2. **Start with previous responders.** Past success is the strongest within-month signal (~3× odds).
3. **Shift to mobile.** Route landline-only, low-score customers to cheaper channels.
4. **Cap repeat calls** at the marginal break-even point (~3–5 attempts in a normal year).
5. **Monitor monthly:** AUC for ranking decay; calibration for level drift.

## License

[MIT](LICENSE)
