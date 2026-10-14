# 🦠 COVID-19 in India — Exploratory Data Analysis

An end-to-end, **executed** Jupyter notebook that analyses three state-wise COVID-19 datasets for
India (cases, testing, vaccination) from **30-Jan-2020 to 16-May-2021**, covering the first and
second waves. It follows the structure of a professional data-science deliverable: introduction →
questions → exploration → cleaning → descriptive statistics → univariate, bivariate & multivariate
visual analysis → time-series → statistical testing → insights.

> **Main deliverable:** [`COVID19_India_EDA.ipynb`](COVID19_India_EDA.ipynb) — open it in Jupyter/
> VS Code. **All outputs (tables, printed statistics and figures) are already saved in the file**,
> so it can be read as a report without re-running anything.

---

## 📁 Project structure

```
COVID DATA ANALYSIS/
├── COVID19_India_EDA.ipynb        # ⭐ the main notebook (WITH executed outputs)
├── README.md                      # this file
├── build_notebook.py              # reproducible generator that builds the .ipynb
├── covid_19_india.csv             # input: confirmed / cured / deaths (state-wise, daily)
├── StatewiseTestingDetails.csv    # input: samples tested (state-wise)
├── covid_vaccine_statewise.csv    # input: vaccination (state-wise + India aggregate)
├── COVID19_Data.zip               # original uploaded archive (contains the same 3 CSVs)
└── plots/                         # all 15 figures exported as high-res PNGs
    ├── 01_national_cumulative_and_daily.png
    ├── 02_daily_new_cases_distribution.png
    ├── 03_boxplots_daily_metrics.png
    ├── 04_state_level_distributions.png
    ├── 05_correlation_national_daily.png
    ├── 06_correlation_state_features.png
    ├── 07_scatter_tests_vs_cases.png
    ├── 08_pairplot_state_features.png
    ├── 09_waves_deaths_active.png
    ├── 10_top_states_daily_new_cases.png
    ├── 11_monthly_new_cases_heatmap.png
    ├── 12_testing_trend.png
    ├── 13_vaccination_progress.png
    ├── 14_vaccination_states_gender.png
    └── 15_lag_correlation_tests_cases.png
```

---

## 🗂️ The datasets

| File | Rows × Cols | Date range | Granularity | Key columns |
|---|---|---|---|---|
| `covid_19_india.csv` | 14,978 × 9 | 2020-01-30 → 2021-05-16 | State × day | `Confirmed`, `Cured`, `Deaths` (cumulative) |
| `StatewiseTestingDetails.csv` | 13,300 × 5 | 2020-04-01 → 2021-05-15 | State × day | `TotalSamples`, `Negative`, `Positive` (cumulative) |
| `covid_vaccine_statewise.csv` | 4,440 × 18 | 2021-01-16 → 2021-05-15 | State × day | doses, gender, age bands, vaccine type, AEFI |

All three are **cumulative counters**, so the analysis converts them to **daily increments** before
any comparison.

---

## 🔍 Questions the analysis answers

1. How did the pandemic evolve nationally, and how many distinct **waves** occurred?
2. Which **states** drive the national burden — is it concentrated or broad-based?
3. Is **testing volume** associated with reported cases, and does weak testing inflate the CFR?
4. How did the **vaccination** campaign progress, and who was covered?
5. What are the **multivariate** relationships between testing, cases, deaths and vaccination?

---

## 🧹 Data quality issues found & handled

The notebook documents these explicitly (Section 5 & 6):

- **Duplicate / inconsistent state names:** `Telengana` vs `Telangana`; `Daman & Diu` vs the merged
  UT `Dadra and Nagar Haveli and Daman and Diu` → standardised & merged.
- **Non-geographic buckets** `Unassigned` and `Cases being reassigned to states` → removed.
- **Sentinel strings** (`'-'`, `' '`) inside numeric columns (`ConfirmedIndianNational`,
  `Negative`) → coerced to `NaN` and cast to numeric.
- **Mixed date formats:** vaccination uses `DD/MM/YYYY`, the others use `YYYY-MM-DD` → parsed
  explicitly (with `dayfirst=True` for vaccination).
- **Heavy missingness in testing:** `Positive` missing in ~59% of rows; the `Positive` column stops
  being reported by most states by Aug–Sep 2020 → positivity is computed by **aligning `Positive`
  and `TotalSamples` to the same reporting day**, with the limitation clearly stated.
- **Vaccination age-band / AEFI columns** are only populated from **16-Mar-2021** → noted, not
  over-interpreted.
- **Cumulative-only reporting** → all metrics are converted to daily increments (clipped at 0 to
  absorb reporting corrections).

---

## 🛠️ Methodology

- **Cleaning:** reusable, side-effect-free functions (`clean_cases`, `clean_testing`, `clean_vax`).
- **Feature engineering:** `Active = Confirmed − Cured − Deaths`; daily new cases/deaths/tests;
  7-day moving averages; CFR %, recovery %, positivity %, tests-per-case, doses.
- **National aggregates:** forward-fill each state before summing so a state pausing reporting does
  not artificially reduce the national cumulative total.
- **Univariate analysis:** histograms (linear + log), KDE, box plots, bar charts.
- **Multivariate analysis:** correlation heat-maps, scatter with regression lines, pair plot.
- **Time series:** wave curves, deaths (lagged), active burden, per-state small multiples, monthly
  heat-map, testing & vaccination trends.
- **Statistics:** Pearson & Spearman correlations, **lagged cross-correlation** (testing vs cases),
  Welch t-test + Mann-Whitney U (Wave 1 vs Wave 2), Cohen's d, and log-linear **doubling-time**
  estimation.

---

## 💡 Key findings (one-line summary)

- **Two waves.** Wave 1 peaked at **≈ 98k cases/day (17-Sep-2020)**; Wave 2 peaked at
  **≈ 414k cases/day (07-May-2021)** — **~4× larger** and, crucially, **~2× faster**
  (doubling time **~12 days vs ~26 days**). Welch t = 7.18, p ≈ 9.7 × 10⁻¹¹.
- **Concentrated but broadening.** Top-5 states ≈ **half** of all cases (Maharashtra, Karnataka,
  Kerala, UP, Tamil Nadu), yet the 2021 wave swept many states simultaneously.
- **Testing was reactive.** Daily tests peaked on the *same day* as cases (07-May-2021); lagged
  cross-correlation peaks at lag ≈ 0. Tests and cases are strongly rank-correlated (Spearman
  ≈ 0.94).
- **CFR is not explained by testing intensity** here (positivity vs CFR r ≈ −0.05, p ≈ 0.79;
  tests-per-case vs CFR r ≈ −0.17, p ≈ 0.31 — insignificant).
- **Vaccination:** from **16-Jan-2021** to **≈ 17.95 crore doses by 14-May-2021**; near-parity
  gender split (≈ 52.6% male / 47.3% female / 0.01% transgender); second doses lagged first doses.
- **Normalisation matters:** all *scale* variables correlate strongly (they proxy state size);
  *ratio* variables do not — ratios are where insight hides.

---

## ▶️ How to run

The notebook is **already executed** — just open it to read the results. To re-run it yourself:

```bash
# 1. (optional) create an environment and install dependencies
pip install pandas numpy matplotlib seaborn scipy jupyter

# 2. launch Jupyter and open the notebook
jupyter notebook COVID19_India_EDA.ipynb
#    then: Kernel -> Restart & Run All
```

**Requirements:** Python 3.10+ with `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, `jupyter`.

**Regenerating the notebook from scratch** (it is built programmatically so it stays reproducible):

```bash
python build_notebook.py
jupyter nbconvert --to notebook --execute --inplace COVID19_India_EDA.ipynb
```

Running it top-to-bottom re-creates every table, statistic and figure, and re-writes the 15 PNGs
into `plots/`.

---

## 📌 Notes & limitations

- No **population** column is present, so per-capita rates could not be computed — volume
  comparisons conflate population size with epidemic intensity. Merging Census/NFHS population data
  is the top recommended next step.
- Testing coverage is uneven and `Positive` reporting shrinks over time, so positivity figures are
  **trend indicators**, not exact national rates.
- Correlations are **not** causal; the CFR–testing result is a hypothesis, not a finding.
- The `India` aggregate row in the vaccination file is excluded from state-level work to avoid
  double counting.

---

## 🚀 Suggested next steps

1. Join **state population** → per-100k incidence, per-1000 testing, per-capita mortality.
2. Add **mobility / vaccination-coverage** regressors and fit a panel model of new cases.
3. Fit **logistic / SIR-style** curves per state to estimate Rt and wave timing.
4. Build a **7-day-ahead forecasting** baseline and a small **dashboard** (the `plots/` folder is
   directly reusable).

---

*Built as a portfolio-quality EDA. Every code cell is followed by a plain-language
**📝 Observation** note so the notebook doubles as a report.*
