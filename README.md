# GlucoseIQ
**End-to-end Diabetes Patient Analytics powered by real UCI + Kaggle clinical data and 4 live CMS Medicare Tier 1 APIs — MySQL · Python · Scikit-learn · Plotly**

---

## Project Overview

GlucoseIQ is a full-stack healthcare data analytics project that combines real patient-level clinical datasets with live U.S. government Medicare APIs to build a 6-table EHR database, run 10 SQL business questions, train two ML models, and produce a 12-chart visualization dashboard — all inside a single Jupyter notebook.

Patient identity data is fully HIPAA-compliant (Safe Harbor method): no names, no dates of birth, no SSNs, no MRNs, no addresses. Clinical features (HbA1c, glucose, BMI, insulin, blood pressure) come from real public research datasets. Drug, hospital, and population benchmark data pull live from CMS Medicare APIs at runtime.

---

## Project Name Rationale

| Name | Taken by |
|------|----------|
| **ClinicalPulse** | Your other project — Cardiovascular, Endocrine & Mental Health |
| **GlucoseIQ** | This project — Diabetes-focused, signals both the clinical biomarker and the analytics intelligence layer |

---

## Data Sources

### Patient-Level Clinical Data (individual rows)

| Source | Method | Patients | Key Fields |
|--------|--------|----------|------------|
| UCI Pima Indians Diabetes (NIDDK 1988) | `fetch_ucirepo(id=34)` | 768 | glucose, insulin, BMI, blood pressure, diabetes outcome |
| Kaggle Diabetes Prediction Dataset | `pd.read_csv()` | 100,000 | HbA1c, glucose, BMI, smoking history, diabetes outcome |

### CMS Medicare Tier 1 APIs (live · free · no login required)

| # | API | Endpoint | What It Provides |
|---|-----|----------|-----------------|
| C | CMS Part D Prescribers | `data.cms.gov/resource/tau9-gfsr.json` | Real diabetes drug prescriptions, claim volumes, costs by provider |
| D | CMS Hospital Quality | `data.cms.gov/provider-data/api/1/datastore/query/xubh-q36u/0` | Real hospital star ratings, 30-day readmission rates, 4,000+ hospitals |
| E | CMS Inpatient DRG | `data.cms.gov/resource/tcsp-6e99.json` | Real average LOS and payments per diabetes DRG code |
| F | CMS Chronic Conditions | `data.cms.gov/resource/cng4-92f3.json` | Real diabetes prevalence by US state for Medicare beneficiaries |

All 4 CMS API calls are wrapped in a `cms_get()` error handler — the notebook runs fully even if an endpoint is temporarily unavailable.

---

## HIPAA Compliance & Data Transparency

This project follows the **HIPAA Safe Harbor method** for de-identification. None of the 18 PHI identifiers are present:

- No patient names
- No dates of birth (age stored as a numeric value only)
- No SSNs, MRNs, or account numbers
- No addresses, zip codes, or geographic data below state level
- No phone numbers, email addresses, or IP addresses
- `patient_id` is a generated row index (`PID000001`) with no link to any real individual

### Data Transparency — Three Tiers

| Column | Source | Type |
|--------|--------|------|
| HbA1c, glucose, BMI, insulin, blood pressure | UCI Pima + Kaggle datasets | ✅ Real clinical measurements |
| Drug names, claim volumes, drug costs | CMS Part D API | ✅ Real Medicare data |
| Hospital names, star ratings, readmission rates | CMS Hospital Quality API | ✅ Real Medicare data |
| Diabetes prevalence by state | CMS Chronic Conditions API | ✅ Real Medicare data |
| `los_days`, `is_readmitted` | `np.random.normal` seeded to CMS DRG benchmarks | ⚠️ Statistically derived |
| `primary_diagnosis` ICD codes | Rule-assigned from diabetes flag | ⚠️ Derived, not from real claims |

> **Portfolio disclosure:** LOS and readmission outcomes are statistically derived using CMS Medicare DRG benchmark distributions. All clinical features (HbA1c, glucose, BMI, insulin) are from real public datasets.

---

## Database Schema — 6-Table EHR in MySQL

```
diabetes_ehr_db
├── Patients           ← UCI + Kaggle (demographics, diagnosis, risk score)
├── LabResults         ← UCI + Kaggle (HbA1c, blood glucose, insulin, flags)
├── VitalSigns         ← UCI Pima (blood pressure, BMI, skin thickness)
├── Prescriptions      ← CMS Part D API (real drug names, costs, claim data)
├── HospitalBenchmarks ← CMS Hospital Quality API (star ratings, readmission rates)
└── Admissions         ← CMS DRG-benchmarked LOS + readmission flags
```

---

## Project Phases

| Phase | Description | Tools |
|-------|-------------|-------|
| 1 | Environment setup, MySQL install, DB creation | Python, MySQL |
| 2 | 6-source data extraction — UCI + Kaggle + 4 CMS APIs | `ucimlrepo`, `requests`, `pd.read_csv()` |
| 3 | 6-table schema + bulk load into MySQL | SQLAlchemy, `to_sql()` |
| 4 | SQL extraction — multi-table JOINs back to DataFrames | `mysql-connector`, CTEs |
| 5 | Data cleaning — nulls, type coercions, outlier handling | pandas, numpy, missingno |
| 6 | EDA — diabetes distributions, correlations, heatmaps | matplotlib, seaborn |
| 7 | KMeans patient segmentation (4 clusters + PCA) | scikit-learn |
| 8 | Statistical analysis — t-tests, chi-square, ANOVA, Pearson | scipy |
| 9 | 10 Business Questions — SQL KPIs with window functions | CTEs, RANK, NTILE, LAG |
| 10 | 12-chart visualization dashboard + Plotly interactive scatter | matplotlib, seaborn, plotly |
| 11 | ML models — diabetes prediction + readmission prediction | LogisticRegression, ROC-AUC |
| 12 | Clinical insights and recommendations | markdown |
| 13 | Export — CSV files + 11-sheet Excel workbook | pandas, openpyxl |

---

## Business Questions Answered (Phase 9)

1. Diabetes rate by age group
2. Drug class outcomes — readmission rate and avg cost per claim (CMS-enriched)
3. HbA1c tier distribution and readmission risk
4. Triple burden patients — diabetes + hypertension + heart disease
5. Hospital quality benchmarking by state (CMS data)
6. LOS and cost analysis by BMI category
7. Risk category distribution across the patient population
8. Drug prescribing patterns by patient risk tier
9. High-risk patient cohort identification (SQL CTE)
10. Window function — cumulative LOS by risk category

---

## ML Models (Phase 11)

### Model 1 — Diabetes Prediction
- **Algorithm:** Logistic Regression with class balancing
- **Features:** age, BMI, HbA1c, blood glucose, insulin, hypertension, heart disease, blood pressure
- **Top predictors:** HbA1c > blood glucose > BMI > age > hypertension

### Model 2 — Readmission Prediction
- **Algorithm:** Logistic Regression with class balancing
- **Features:** age, BMI, HbA1c, blood glucose, hypertension, heart disease, diabetes, LOS, comorbidity count
- **Top predictors:** comorbidity count > diabetes > HbA1c > LOS > hypertension

Both models output confusion matrices, classification reports, ROC-AUC scores, and feature coefficient charts.

---

## Tech Stack

```
Language     Python 3.x
Database     MySQL 8 (local via subprocess)
APIs         requests — 4 live CMS Socrata REST endpoints
             ucimlrepo — UCI ML Repository API
Data         pandas, numpy
Viz          matplotlib, seaborn, plotly
ML           scikit-learn (KMeans, PCA, LogisticRegression, ROC-AUC)
Stats        scipy (t-test, chi-square, ANOVA, Pearson)
DB layer     SQLAlchemy, mysql-connector-python
Export       openpyxl (11-sheet Excel), CSV
```

---

## Outputs

| File | Contents |
|------|----------|
| `patients_master.csv` | Full merged patient table |
| `prescriptions_cms_partd.csv` | CMS drug data mapped to patients |
| `hospital_benchmarks_cms.csv` | Real hospital quality data |
| `cms_diabetes_by_state.csv` | State-level prevalence from CMS |
| `diabetes_dashboard.png` | 12-chart static dashboard |
| `plotly_diabetes_scatter.html` | Interactive HbA1c vs glucose scatter |
| `ml_models.png` | Confusion matrices + feature coefficients |
| `Diabetes_Analytics_CMS_Report.xlsx` | 11-sheet Excel workbook |

---

## Setup

```bash
# Install dependencies (notebook handles this automatically in Phase 1)
pip install mysql-connector-python sqlalchemy pandas numpy matplotlib seaborn \
            plotly scipy statsmodels scikit-learn missingno tabulate \
            openpyxl ucimlrepo requests

# For Kaggle dataset (optional — notebook has 2 fallbacks if unavailable)
kaggle datasets download -d iammustafatz/diabetes-prediction-dataset --unzip
```

Run all cells top-to-bottom. MySQL is installed and configured automatically in Phase 1.

---

## Repo Structure

```
glucoseiq/
├── Diabetes_Analytics_CMS_Tier1.ipynb   ← Main notebook (all 13 phases)
├── diabetes_prediction_dataset.csv       ← Kaggle dataset (if pre-downloaded)
├── README.md
└── outputs/
    ├── diabetes_dashboard.png
    ├── plotly_diabetes_scatter.html
    ├── ml_models.png
    ├── Diabetes_Analytics_CMS_Report.xlsx
    └── *.csv
```

---

## Related Project

**ClinicalPulse** — MySQL-powered Cardiovascular, Endocrine & Mental Health analytics using real CMS Medicare data with HIPAA-compliant synthetic patients.
