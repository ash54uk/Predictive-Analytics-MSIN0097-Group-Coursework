# Interaction Log

**Date:** 13 March 2025  
**Project:** Manchester Property Price Prediction — Predictive Analytics

---

## Interaction 1: End-to-End Data Science Notebook

### Request
Create a single, end-to-end Jupyter notebook covering the full data science pipeline for the Manchester property dataset (`manchester_features.parquet`):

1. Data Ingestion & Cleaning  
2. Exploratory Data Analysis  
3. Predictive Modelling  
4. Model Improvement  

### Deliverables
- **`manchester_property_analysis.ipynb`** — Complete pipeline notebook
- **`requirements.txt`** — Dependencies (pandas, pyarrow≥19.0.1, numpy, scikit-learn, matplotlib, seaborn)

### Key Implementation Details
- Loaded parquet dataset (474,144 rows, 23 columns)
- Cleaning: dropped identifiers, uninformative columns; removed price outliers (£10k–£2M); imputed floor_area; dropped epc_rating/co2_emissions (high missingness)
- EDA: price distributions, property type/borough analysis, temporal trends, correlation heatmap
- Baseline model: Random Forest with OneHotEncoder and StandardScaler
- Improved model: Gradient Boosting, log(price) target, town feature, floor_area_ratio
- All randomness set to `random_state=42` for reproducibility

### Technical Note
PyArrow 19.0.0 had a known bug ("Repetition level histogram size mismatch"); upgraded to 19.0.1+ to read the parquet file.

---

## Interaction 2: Bug Fixes in `broken_pipeline.ipynb`

### Request
Find and fix all bugs preventing the notebook from running correctly or producing invalid results.

### Bugs Found and Fixed

| # | Bug | Impact | Fix |
|---|-----|--------|-----|
| 1 | `mean_squared_error` used but not imported | `NameError` — notebook crashes at evaluation | Added `mean_squared_error` to sklearn.metrics import |
| 2 | Target leakage: `X = df.copy()` included `price` | Model sees the answer; artificially inflated R² | Changed to `X = df.drop(columns=['price']).copy()` |
| 3 | Wrong column `property_type_raw` (does not exist) | `KeyError` — pipeline fails at encoding | Changed to `property_type` (correct column name) |
| 4 | Scaler fitted on full data before train/test split | Data leakage — test statistics influence training | Split first, then `fit_transform` on train only, `transform` on test |

### Outcome
Notebook runs end-to-end and produces valid evaluation metrics (RMSE, MAE, R²) without leakage.

---

## Interaction 3: Create Interaction Log

### Request
Create a markdown file log of this interaction.

### Deliverable
This file: **`INTERACTION_LOG.md`**

---

## Files in Project

| File | Description |
|------|--------------|
| `manchester_features.parquet` | Source dataset (Greater Manchester property transactions, 2015–2024) |
| `manchester_property_analysis.ipynb` | Full pipeline notebook (ingestion → cleaning → EDA → modelling → improvement) |
| `broken_pipeline.ipynb` | Simplified ML pipeline (bug-fixed) |
| `requirements.txt` | Python dependencies |
| `INTERACTION_LOG.md` | This log |
