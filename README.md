# PharmaSafeIQ: FDA Adverse Events Analysis (2015–2025)

An end-to-end data analytics project on **528,000 FDA adverse event reports**. Python for cleaning, SQLite and SQL for business analysis, and a 2-page Power BI dashboard for drug, manufacturer and country risk.

**Tools:** Python (pandas, NumPy, Matplotlib, seaborn) · SQLite / SQL (CTEs, window functions) · Power BI (DAX)

---

## Business Problem

A drug safety team needs to know where the highest-risk signals are in adverse event reporting:

1. Which drugs have the highest fatality rate?
2. Which manufacturers are riskier than the industry average?
3. Do polypharmacy (many drugs) and patient age combine to raise fatality?
4. Is the share of serious and fatal reports improving over time?
5. How do countries compare in report volume and fatality?

---

## Dataset

| Item | Detail |
|---|---|
| File | `fda_adverse_events_2015_2026_CLEAN.csv` |
| Rows | 528,000 reports (one row per `report_id`) |
| Period | 2015–2025 (exactly 48,000 reports per year, so the data is a sample, not full FAERS) |
| Columns | 30 (report date, seriousness flags, fatal/hospitalized/life-threat/disabling flags, reactions, suspect drug, brand, route, indication, manufacturer, pharma class, number of drugs, patient age/sex/weight, country) |

---

## Tools & Technologies

| Category | Tool | Used for |
|---|---|---|
| Language | **Python** | Data loading, cleaning, validation, feature checks |
| Libraries | **pandas, NumPy** | Missing value handling, outlier checks, aggregation, CSV export |
| Visualisation (Python) | **Matplotlib, seaborn** | Charts for each SQL question inside the notebook |
| Database | **SQLite** | Storing the cleaned `adverse_events` table (`PharmaSafeIQ.db`) |
| Query language | **SQL** | CTEs, `CROSS JOIN`, `CASE` bucketing, aggregations, `RANK() OVER()` window function |
| Environment | **Jupyter Notebook** | Step-by-step analysis with an insight written under every output |
| BI tool | **Power BI Desktop** | 2-page interactive dashboard |
| Power BI language | **DAX** | Measures such as Top Risk Drug, Fatal Rate %, Serious Rate % and Peak Fatal Year |

---

## Project Workflow

```
Raw CSV  →  Python cleaning  →  SQLite (PharmaSafeIQ.db)  →  5 SQL business questions  →  CSV export  →  Power BI dashboard
```

### 1. Data Quality & Cleaning (Python)

- **Duplicates:** 0 full-row duplicates and 0 duplicate `report_id`s, so no removal was needed.
- **Missing values:**
  - `patient_weight_kg`: 71.96% missing. Dropped because imputing it would add bias and no business question uses it.
  - `serious_flags`: 57.03% missing. Filled with `'Unknown'` instead of dropping 57% of rows.
  - `patient_age_years`: 28.69% missing. Kept in the data and excluded only from the age-based analysis (Q3).
- **Outliers:** `num_drugs` has a median of 4 but a mean of 8.70 and a max of 2,108 (right-skewed). Outliers were kept because they may be real reports. Drug counts were bucketed (1–4, 5–9, 10+) so extreme values don't distort results.
- **Validation:** `num_drugs` counts all drugs in a report, while `suspect_drug` counts only suspect drugs, so the two columns measure different things. `num_drugs` is used in the analysis.

### 2. SQL Analysis (SQLite)

Cleaned data was loaded into `adverse_events` and queried with aggregations, CTEs, `CROSS JOIN`, `CASE` bucketing and `RANK() OVER()`.

### 3. Power BI Dashboard

The query results were turned into 5 tables (`Drug_Risk`, `Manufacturer_Risk`, `Country_Analysis`, `Yearly_Trends`, `Polypharmacy`) and visualised on two report pages, with DAX measures such as *Top Risk Drug*, *Top Risk Manufacturer*, *Top Country by Reports*, *Peak Fatal Year*, *Fatal Rate %* and *Serious Rate %*.

---

## Key Insights

The overall fatality rate across all reports is **10.28%**.

### Q1. Drugs with the highest fatality rate (min. 500 reports)
- The top 10 drugs have fatality rates of **34.03% to 47.71%**.
- **Pimavanserin Tartrate** is highest (1,605 fatal of 3,364 reports = 47.71%), followed by **Sitagliptin** at 46.93%.
- **Rivaroxaban** has the most reports in the top 10 (12,968), with a 38.81% fatal rate and a 91.58% hospitalization rate.
- Pimavanserin's hospitalization rate is only 31.60%, so its reports skew towards fatal outcomes rather than hospitalization.

### Q2. Manufacturers above the industry average (min. 500 reports)
- The top 10 manufacturers sit at **24.80% to 47.67%**, about **2.4x to 4.6x** the industry average.
- **Acadia Pharmaceuticals** is first at 47.67%. Its 1,605 fatal reports are the same ones behind Pimavanserin in Q1, so its rank comes entirely from one drug.
- Next are **Regeneron** (32.25%) and **Merck Sharp & Dohme** (31.86%). **Actelion** has high volume (4,398 reports, 1,164 fatal) at 26.47%.

### Q3. Polypharmacy × age
| Age group | 1–4 drugs | 5–9 drugs | 10+ drugs |
|---|---|---|---|
| Below 60 | 6.45% | 7.46% | 9.04% |
| 60+ | 14.78% | 15.12% | 16.80% |

- Fatality rises with drug count in **both** age groups: +2.59 points (below 60) and +2.02 points (60+) from 1–4 to 10+ drugs.
- Age has the bigger effect. At 1–4 drugs, patients aged 60+ are already at **2.3x** the fatality rate of younger patients.
- The highest-risk group is **60+ with 10+ drugs (16.80%)**.

### Q4. Serious and fatal reports over time
- Fatal rate fell from **10.91% (2015) to 7.53% (2024)**, with a small rise to 8.47% in 2025.
- **2020 is a major outlier**: 8,512 fatal reports (17.73%), against 3,613 to 5,237 in other years.
- The serious rate does not follow a straight trend: 78.24% (2015), a low of 68.40% (2017), 81.36% (2020), and a high of **82.00% (2025)**.
- Because every year has exactly 48,000 reports, only the rates (not the counts) should be compared.

### Q5. Country-wise reports and fatality (min. 100 reports, top 15)
- The **US** has 317,456 reports (60.1% of the data) with a 7.98% fatal rate.
- **Japan** has the highest fatal rate at **23.77%** (6,011 of 25,287), followed by France (17.32%), Great Britain (16.66%) and the Netherlands (15.90%).
- The US serious rate is **59.89%**, while all other top countries are at **96.51% to 99.07%**. This likely reflects reporting rules (foreign reports are mostly submitted for serious cases), so countries should not be ranked as "more dangerous" from this alone.

---

## Limitations

- Fatality rate here is the **share of reports that are fatal**, not true drug risk. The dataset has no denominator (how many people take the drug), so these are reporting-based signals, not causal findings.
- The dataset is a **sample** (exactly 48,000 reports per year), not the full FDA database.
- Some `suspect_drug` and `manufacturer` values are combinations joined with `;`.
- Country comparisons are affected by differences in reporting rules.

---

## Repository Structure

```
├── fda_adverse_events_analysis.ipynb     # Python cleaning + SQLite + 5 SQL questions
├── PharmaSafeIQ_Dashboard.pbix           # Power BI dashboard (2 pages)
├── fda_adverse_events_2015_2026_CLEAN.csv  # Source data (add locally)
├── fda_clean_for_powerbi.csv             # Cleaned export used in Power BI
├── screenshots/                          # Dashboard screenshots
└── README.md
```

## How to Run

```bash
pip install pandas numpy matplotlib seaborn jupyter
jupyter notebook
```

1. Place `fda_adverse_events_2015_2026_CLEAN.csv` in the same folder as the notebook.
2. Run all cells. This creates `PharmaSafeIQ.db` and `fda_clean_for_powerbi.csv`.
3. Open the `.pbix` file in Power BI Desktop to explore the dashboard.

## Dashboard Preview

| Page 1: Safety Trend & Polypharmacy | Page 2: Drug, Manufacturer & Country Risk |
|---|---|
| ![Page 1](screenshots/dashboard_page1.png) | ![Page 2](screenshots/dashboard_page2.png) |

## Author

**Taniya**: Data Analyst · Python · SQL · Power BI · Excel
