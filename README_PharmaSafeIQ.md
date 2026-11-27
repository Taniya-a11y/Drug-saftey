
<h1 align="center"> PharmaSafeIQ: FDA Adverse Events Drug Safety Analysis |</h1>

<p align="center">
  Analysis of 528,000 FDA adverse event reports (2015–2025) using Python, SQL and Power BI to track drug fatality risk, manufacturer risk, the effect of age and polypharmacy, yearly safety trends and country-level reporting patterns.
</p>

---

## 📌 Project Overview

Every year, thousands of adverse events are reported to the FDA after people take medicines. Counting reports is not enough. A drug safety team needs to know which drugs and manufacturers have an unusually high share of fatal outcomes, whether taking many drugs together makes things worse, and whether overall safety is improving. This project uses an FDA adverse events dataset to answer those questions.

The work has three stages:

1. **Python:** load the data, check quality, decide how to handle missing values and outliers.
2. **SQL (SQLite inside Python):** answer five business questions with CTEs, window functions and bucketed analysis.
3. **Power BI:** a 2-page dashboard built on the SQL result tables.

## ❓ Problem Statement

Raw adverse event reports are one row per report, with long drug and manufacturer names, skewed counts and many missing fields. The goal is to turn them into clear safety signals on:

- Drugs with the highest fatality rate
- Manufacturers with a fatality rate above the industry average
- Whether age and the number of drugs taken together raise fatality
- Whether the share of serious and fatal reports is improving over time
- How reports and fatality differ across countries

## 📂 Dataset Description

| Item | Detail |
|---|---|
| Source | FDA adverse events data (`fda_adverse_events_2015_2026_CLEAN.csv`) |
| Period | 2015 – 2025 (11 years) |
| Raw size | 5,28,000 reports (one row per `report_id`) |
| Reports per year | Exactly 48,000, which shows the data is a sample and not the full FAERS database |
| Key columns | report_id, receive_date, year, patient_age_years, patient_sex, serious, serious_flags, is_fatal, is_hospitalized, suspect_drug, num_drugs, num_reactions, manufacturer, country |

`is_fatal` and `is_hospitalized` are 0/1 flags, so `SUM` gives counts and `SUM / COUNT` gives rates.

## 🛠️ Tools & Technologies

| Area | Tools |
|---|---|
| Data cleaning | Python (pandas, NumPy) |
| Exploration | Matplotlib, seaborn |
| Analysis | SQL (SQLite): CTEs, window function (`RANK`), `CASE WHEN` buckets, `CROSS JOIN`, `HAVING` |
| Visualization | Power BI (KPI cards, bar, column, line and combo charts, table, slicers) |
| Environment | Jupyter Notebook |

## ⚙️ Project Workflow

**1. Data loading and quality checks**
- Loaded 528,000 reports and converted `receive_date` to datetime.
- Missing values: `patient_weight_kg` **71.96%** (379,923), `serious_flags` **57.03%** (301,106) and `patient_age_years` **28.69%** (151,509). All other columns have 0% missing.
- 0 full-row duplicates and 0 duplicate `report_id`, so every row is a unique report.
- Patient sex: Female 52.9%, Male 37.7%, Unknown 9.5%. Serious reports: 74.8% Yes, 25.2% No.

**2. Cleaning decisions**
- `serious_flags`: filled missing with `Unknown`, because dropping rows would have lost 57% of the data.
- `patient_weight_kg`: dropped the column. Imputing 72% missing values would add bias and no business question uses weight.
- `patient_age_years`: nulls are kept, but excluded from the age analysis (Q3) only.
- Outliers: average age is 55.94 (median 59). `num_drugs` has a median of 4 but a mean of 8.70 and a maximum of 2,108, and `num_reactions` goes up to 253. These were **not deleted** because they can be real reports. Drug counts were grouped into buckets (1-4, 5-9, 10+) so extreme values do not distort results.
- `num_drugs` counts all drugs in a report, while the `suspect_drug` column only lists suspect drugs. The two differ in 422,426 rows, which is expected and not an error. `num_drugs` is used for analysis.

**3. SQLite setup**
- Loaded the cleaned DataFrame into an `adverse_events` table (528,000 rows, matches the DataFrame).

**4. SQL business analysis (5 questions)**

| # | Business question | Techniques |
|---|---|---|
| Q1 | Which drugs have the highest fatality rate? (min 500 reports) | `RANK() OVER`, `HAVING` |
| Q2 | Which manufacturers are riskier than the industry average? (min 500 reports) | CTEs, `CROSS JOIN` |
| Q3 | Do age and polypharmacy together raise fatality? | Two `CASE WHEN` bucket columns |
| Q4 | Are serious and fatal report shares improving over time? | `GROUP BY year`, conditional `SUM` |
| Q5 | Country-wise reports and fatality (min 100 reports, top 15) | `GROUP BY`, `HAVING` |

**5. Power BI export and dashboard**
- Built a 2-page dashboard on the SQL result tables, with KPI cards, bar, column, line and combo charts, a country table and interactive **slicers** (dropdown): Year on Page 1 and Country on Page 2.

## 💡 Key Insights

Findings from the five SQL questions (528,000 reports, 2015–2025). Fatality rates are shares of reports, not real-world risk (see Limitations).

**Q1: Highest-fatality drugs**
- Overall average fatality rate is **10.28%**. The top 10 drugs range from **34.03% to 47.71%**.
- **Pimavanserin Tartrate** is highest (**47.71%**, 1,605 fatal of 3,364 reports), followed by **Sitagliptin (46.93%**, 260 of 554).
- **Rivaroxaban** has the most reports in the top 10 (**12,968**), with a 38.81% fatality rate and a **91.58%** hospitalization rate. Pimavanserin has only a 31.60% hospitalization rate, so its reports lead to fatal outcomes more directly.

**Q2: Manufacturer risk vs industry average**
- The top 10 manufacturers range from **24.80% to 47.67%**, which is **2.4x to 4.6x** the industry average of 10.28%.
- **Acadia Pharmaceuticals** is highest (**47.67%**, 1,605 fatal of 3,367). These are the same 1,605 fatal reports as Pimavanserin in Q1, so Acadia's rank comes from a single drug.
- Next: **Regeneron (32.25%, 521 reports)** and **Merck Sharp & Dohme (31.86%, 3,544 reports)**. **Actelion** has a 26.47% rate on a large volume (4,398 reports, 1,164 fatal).

**Q3: Age and polypharmacy**

| Drug count | Below 60 | Older (60+) |
|---|---|---|
| 1-4 drugs | 6.45% | 14.78% |
| 5-9 drugs | 7.46% | 15.12% |
| 10+ drugs | 9.04% | 16.80% |

- Fatality rises with the number of drugs in **both** age groups.
- Age matters more than drug count: patients aged 60+ have more than double the fatality rate at the same drug count (14.78% vs 6.45% for 1-4 drugs).
- The highest-risk group is **60+ with 10+ drugs (16.80%)**, about **2.6x** the lowest group (below 60 with 1-4 drugs, 6.45%). Reports with missing age (151,509) are excluded here, leaving 376,491 reports.

**Q4: Yearly trend**
- **Serious rate:** 78.24% (2015), down to 68.40% (2017), up to 81.36% (2020) and **82.00% in 2025**, its highest point.
- **Fatal rate:** 10.91% (2015) down to **7.53% (2024)**, then 8.47% in 2025.
- **2020 is the biggest outlier:** 8,512 fatal reports (**17.73%**), while other years had between 3,613 and 5,237. After 2020 the fatal rate fell from 10.57% (2021) to 7.53% (2024).
- Because every year has exactly 48,000 reports, only the rates are meaningful, not the report counts.

**Q5: Country view**
- The **US** has **317,456 reports (60.1%)** with a fatality rate of 7.98%.
- **Japan** has the highest fatality rate (**23.77%**, 6,011 of 25,287), then France (17.32%), Great Britain (16.66%) and the Netherlands (15.90%). **Canada** has 54,616 reports but only 6.69% fatal.
- The US serious rate is **59.89%**, while all other top countries are at **96.51% to 99.07%**. This most likely reflects reporting rules (foreign reports are mostly submitted for serious cases), so countries should not be ranked as "more dangerous" directly.

## 📊 Dashboard Screenshots

**Page 1: Safety Trend and Polypharmacy**

Cards for Total Reports, Fatal Rate %, Serious Rate % and Peak Fatal Year. Charts: fatality rate by drug count and age group, annual fatal reports trend, and serious vs fatal rate line chart. Filter: Year dropdown slicer.

![Dashboard Page 1](images/dashboard_page1.png)

**Page 2: Drug, Manufacturer & Country Risk**

Cards for Top Risk Drug, Top Risk Manufacturer and Top Country by Reports. Charts: top 10 drugs by fatality rate, manufacturer fatality rate vs industry average, countries by number of reports, and a country-wise risk summary table. Filter: Country dropdown slicer.

![Dashboard Page 2](images/dashboard_page2.png)

## 🗂️ Folder Structure

```
PharmaSafeIQ/
├── fda_fixed.ipynb                                  # cleaning, SQL analysis, CSV export
├── fetal_yaer.pbix                                  # Power BI dashboard (2 pages)
├── fda_adverse_events_2015_2026_CLEAN.csv           # source data
├── PharmaSafeIQ.db                                  # SQLite database
├── outputs/
│   └── powerbi/                                     # SQL result tables used by Power BI
├── images/
│   ├── dashboard_page1.png
│   └── dashboard_page2.png
└── README.md
```

## ▶️ How to Run

1. Put `fda_adverse_events_2015_2026_CLEAN.csv` in the same folder as the notebook.
2. Install the libraries: `pip install pandas numpy matplotlib seaborn`. (`sqlite3` comes with Python.)
3. Run the notebook from top to bottom. It creates `PharmaSafeIQ.db` and loads the `adverse_events` table.
4. Open `fetal_yaer.pbix` in Power BI Desktop and, if asked, point the data source to the folder with the SQL result tables.

## ⚠️ Limitations

- The data is a **sample** (exactly 48,000 reports per year), not the complete FAERS database.
- Fatality rate is the share of **reports** that were fatal. The data has no count of how many people take each drug, so this is a reporting-based signal and not the true risk of a drug.
- Some `suspect_drug` and `manufacturer` values are several names joined by `;`, so combinations are grouped as one entry.
- A manufacturer's rank can come from one drug (Acadia and Pimavanserin share the same 1,605 fatal reports).
- Country comparisons are affected by reporting rules, which is why the US serious rate is far lower than other countries.
- 28.69% of patient ages are missing, and weight (71.96% missing) was dropped.
- Extreme values (up to 2,108 drugs in one report) were kept and handled with buckets, not removed.
- 2020 has an unusually high number of fatal reports, which this data cannot explain.

## 👩‍💻 Author

**Taniya**
Entry-level Data Analyst

- LinkedIn: https://www.linkedin.com/in/taniya-k-826856274
- GitHub: https://github.com/Taniya-a11y
- Email: Taniyakaushal513@gmail.com
