# Diabetes Onset & 30-Day Readmission Risk Analysis

**[Click here to view the full interactive notebook report](https://abhijeethganji99.github.io/GlucoseIQ/)**

End-to-end healthcare analytics project combining real patient-level survey data, real claims-style patient records, and a live CMS public API to predict diabetes onset and 30-day hospital readmission risk.

## Overview

This project builds a MySQL-backed clinical data pipeline, segments patients via unsupervised learning, and trains two independently validated logistic regression models — one predicting diabetes onset on a large real-world population health survey, and one predicting 30-day readmission risk on a patient-level clinical dataset enriched with live CMS hospital quality data.

## Data Sources

| Source | Description | Status |
|---|---|---|
| UCI CDC Diabetes Health Indicators (id=891) | 253,680 real BRFSS survey respondents — health indicators, lifestyle, and demographics | Live, confirmed working |
| Kaggle Diabetes Prediction Dataset | 100,000 patient-level records with lab values, vitals, and admission outcomes | Live, confirmed working |
| CMS Hospital Quality API | Live provider-level hospital quality data | Live, confirmed working |
| CMS Part D Prescribers | Medicare prescriber drug claims | Retired (HTTP 410) — fallback schema used |
| CMS Inpatient DRG | Medicare inpatient length-of-stay and cost data | Retired (HTTP 410) — fallback schema used |
| CMS Chronic Conditions | State-level chronic disease prevalence | Retired (HTTP 410) — fallback schema used |

The diabetes-onset model and the readmission model are trained on two separate datasets rather than one merged table. The CDC survey dataset (categorical health/lifestyle indicators) and the Kaggle dataset (continuous lab values) don't share a compatible schema, so merging them would have meant fabricating overlap that doesn't exist in the source data. Keeping them separate was the more honest choice.

## Pipeline

1. **Environment setup** — MySQL instance, Python ML/stats stack
2. **Data extraction** — UCI CDC891 survey data, Kaggle patient records, CMS Hospital Quality API
3. **Database build** — six normalized tables (Patients, LabResults, VitalSigns, Prescriptions, HospitalBenchmarks, Admissions) loaded via SQLAlchemy
4. **Cleaning** — null/duplicate/outlier detection (Pandas, NumPy, Missingno), group-median imputation, winsorization
5. **SQL analysis** — joins, CTEs, window functions (RANK, NTILE) to answer cohort-level business questions
6. **Unsupervised segmentation** — KMeans clustering (k selected via elbow method and silhouette score), PCA for visualization
7. **Predictive modeling** — logistic regression for both diabetes onset and readmission risk, evaluated via ROC-AUC, confusion matrices, and classification reports
8. **Visualization** — 12-panel dashboard plus supporting EDA and correlation charts (Matplotlib, Seaborn, Plotly)

## Results

**Diabetes onset prediction** (UCI CDC891, 253,680 patients)
- ROC-AUC: 0.815
- Accuracy: 72.8%
- Trained on 202,944 patients, tested on 50,736
- Features: age, BMI, hypertension, high cholesterol, heart disease, smoking status, stroke history, physical activity, general/mental/physical health ratings, difficulty walking

**30-day readmission prediction** (Kaggle-based patient table, 100,000 patients)
- ROC-AUC: 0.672
- Accuracy: 90.0%
- Trained on 80,000 patients, tested on 20,000
- Features: age, BMI, HbA1c, blood glucose, hypertension, heart disease, diabetes status, length of stay, comorbidity count

A note on the readmission accuracy figure: the real-world readmission rate in this dataset is 7.5%, meaning a model that predicted "never readmitted" for every patient would score 92.5% accuracy without learning anything. The 90.0% figure is reported for completeness, but ROC-AUC (0.672) is the metric that actually reflects this model's genuine, above-chance predictive signal on an imbalanced outcome.

**Patient segmentation**
- 4 KMeans clusters (selected via silhouette score)
- 53.6% variance explained by the first 2 PCA components

## Repository Structure

```
notebook.ipynb              full pipeline, end to end
diabetes_dashboard.png      12-panel patient analytics dashboard
ml_models.png               confusion matrices + feature coefficients for both models
clustering_elbow.png        KMeans elbow/silhouette diagnostics
clustering_pca.png          PCA-projected patient segments
eda_overview.png            population-level distributions
eda_correlation.png         clinical feature correlation heatmap
```

## Stack

Python, MySQL, SQLAlchemy, Pandas, NumPy, Scikit-learn, SciPy, Matplotlib, Seaborn, Plotly, Missingno

## Notes on Reproducibility

This pipeline was run end-to-end with full internet access (Google Colab) to fetch live data from UCI and the CMS Hospital Quality API. Running it in a network-restricted environment will cause the UCI fetch to fail and the model results to fall back to synthetic data, which will not reproduce the metrics reported above. The three retired CMS endpoints will return HTTP 410 regardless of environment, by design of the fallback logic.
