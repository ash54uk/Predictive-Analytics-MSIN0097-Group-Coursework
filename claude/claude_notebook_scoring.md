# Claude Code — Main Notebook Scoring

**Tool:** Claude Code (VS Code extension)  
**Task:** End-to-end data science notebook from shared prompt  
**Output:** `manchester_property_analysis.ipynb` (76 cells)  
**First-attempt success:** No  
**Total interactions:** 3 (notebook generation, parquet loading debug, savefig fix)  

---

## Execution Summary

| # | User Message | Claude Code Response | Resolved |
|---|-------------|---------------------|----------|
| 1 | Standard benchmark prompt — produce end-to-end notebook | Explored dataset from command line (schema, dtypes, missing values, cardinalities, price distribution), then generated complete 76-cell notebook | Y |
| 2 | `OSError: Repetition level histogram size mismatch` when running load cell | Investigated across multiple exchanges. Tested both pyarrow and fastparquet engines from terminal. Discovered `manchester_features.parquet` is a nested Spark-partitioned directory (11 Snappy part files), not a single file. Kernel was never restarted between package installs, keeping stale pyarrow active. Fixed by pointing `pd.read_parquet()` at inner directory with `engine='fastparquet'`. | Y — after multiple sub-attempts |
| 3 | `NameError: name 'savefig' is not defined` | Identified `savefig` helper accidentally dropped from cell 1 during earlier edits. Restored it. Notebook runs end-to-end cleanly. | Y |

**Root cause of failures:** The parquet "file" was actually a nested Spark-partitioned directory, which Claude Code did not recognise upfront. Additionally, the Jupyter kernel was never restarted between package installations, so the stale pyarrow version remained active in memory regardless of upgrades.

---

## Final Model Results

| Model | MAE | RMSE | R² | MdAPE |
|-------|-----|------|----|-------|
| Ridge (baseline) | £60,340 | £95,776 | 0.52 | 22.8% |
| Random Forest | £40,237 | £70,684 | 0.75 | 13.7% |
| XGBoost | £41,237 | £71,272 | 0.75 | 14.4% |

**Key findings:** Location (lat/lon) is the dominant price driver. New-build premium = 27.6%. Floor area Pearson r = 0.44. Transaction volume dipped in 2020 (COVID) and peaked in 2021 (stamp duty holiday). Trafford and Stockport command median prices 50–70% above Oldham and Rochdale.

**Dataset:** 474,144 transactions | **Cleaned rows:** 464,689 | **Notebook cells:** 76

---

## Part A: Data Ingestion & Cleaning (27/30)

| Criterion | Score | Justification |
|-----------|-------|---------------|
| Schema exploration | 4 | Printed dtypes, missing counts with percentages, unique-value counts for low-cardinality columns, and commentary. Column key table provided. |
| Uninformative column detection | 5 | Detected `nearest_school_ofsted` is entirely "Not available (OSM source)" despite zero nulls. Dropped 8 columns total (`tx_id`, `postcode`, 3 name fields, `nearest_school_ofsted`, `district`, `town`) — each with per-column justification. |
| Missingness handling | 5 | Recognised `epc_rating`, `floor_area`, and `co2_emissions` are missing for the exact same 33.5% of rows (structural missingness — properties without EPC certificates). Created `has_epc` binary indicator to preserve the missingness signal. Median imputation deferred to modelling Pipeline. |
| Outlier handling | 5 | Flagged £1 and £292M prices specifically, explained what they represent (admin transfers, commercial). Used 1st/99th percentile clipping (removed 1.99% of rows) with justification. Also capped `floor_area` extremes at 1st/99th percentiles. |
| Date handling | 5 | Parsed `transfer_date` into year, month, quarter features. Later added cyclical sin/cos encoding for month in feature engineering. |
| Documentation | 3 | Markdown justification cells for every major decision. Good overall, but some sections like floor area clipping have minimal explanation. |

---

## Part B: Exploratory Data Analysis (19/25)

| Criterion | Score | Justification |
|-----------|-------|---------------|
| Plot variety | 4 | 9 plots: price distribution (raw + log), borough median bar, property type box, quarterly price trend, floor area scatter, EPC rating box, new-build premium comparison, correlation heatmap, transaction volume by year. |
| Plot quality | 4 | Formatted axes with £k labels, titles, colourbars, appropriate sizing throughout. Publication-ready quality. |
| Insight depth | 4 | Specific quantified insights: Trafford/Stockport premium 50–70% above Oldham/Rochdale, new-build premium 27.6%, COVID volume dip 2020, stamp duty surge 2021, floor area Pearson r = 0.44. |
| Feature relationships | 4 | Bivariate analysis throughout: floor area vs price, EPC rating vs price, property type vs price. Correlation matrix covers all numeric features. |
| Handling of cleaning choices | 3 | Used `has_epc` filter for EPC-specific plots. However, **no spatial map** using lat/lon coloured by price — a key differentiator the rubric specifically flags. |

> **Key miss:** No spatial/geographic visualisation despite lat/lon being available and being the dominant predictor in the final model.

---

## Part C: Predictive Modelling (26/30)

| Criterion | Score | Justification |
|-----------|-------|---------------|
| Model choice | 5 | Ridge regression as baseline on log-transformed price (`log1p`), with clear justification: linear, interpretable, establishes performance floor. Log transform explained as reducing skew and penalising proportional errors. |
| Feature selection | 4 | Dropped all IDs and high-cardinality columns. Kept distance features, coordinates, temporal features. 80/20 split stratified by borough. |
| Encoding strategy | 5 | Ordinal encoding for EPC rating (A→G natural order). OneHot for property type, old/new, duration, borough. ColumnTransformer with separate pipelines for numeric (median impute + scale), OHE, and ordinal features. |
| Data leakage prevention | 5 | sklearn Pipeline with ColumnTransformer used throughout — imputation, scaling, and encoding fitted only on training data. Train/test split done before any transformation. |
| Evaluation | 4 | MAE, RMSE, R², and MdAPE reported. Actual vs predicted scatter plots and residual analysis included. Visual diagnostics present. |
| Interpretation | 3 | Explained metrics and identified underprediction at high prices. Could go deeper on why Ridge specifically struggles (R² = 0.52) and what nonlinear interactions are missing. |

---

## Part D: Model Improvement (22/30)

| Criterion | Score | Justification |
|-----------|-------|---------------|
| Strategy variety | 4 | Three distinct strategies: feature engineering (cyclical month, total distance) + Random Forest + XGBoost with tuned hyperparameters. |
| Feature engineering | 3 | Cyclical month encoding (sin/cos) and total proximity score (sum of station, school, supermarket distances). Decent but limited — no price per sqm, no postcode district aggregates, no interaction terms. |
| Improvement magnitude | 4 | R² improved from 0.52 (Ridge) to 0.75 (Random Forest/XGBoost). MAE dropped from £60k to £40k. Substantial improvement. |
| Fair comparison | 5 | Same test set used throughout. Side-by-side metrics table (MAE, RMSE, R², MdAPE) and comparison bar chart. Three models compared quantitatively. |
| Leakage awareness | 4 | No target leakage. Pipeline used consistently. Could explicitly discuss why certain features are/aren't safe. |
| Honesty | 4 | Honest reporting — noted RF and XGBoost performed similarly (both R² = 0.75). Didn't overstate XGBoost gains. Suggested further improvements (Optuna tuning, spatial features, stacking). |

> **Key miss:** Feature engineering was the weakest area. No price-per-sqm, no postcode-district aggregates, no borough × property_type interactions. RF and XGBoost converged at similar performance (both R² = 0.75), suggesting the feature set — not the algorithm — is the bottleneck.

---

## Overall Score

| Section | Score |
|---------|-------|
| A. Data Ingestion & Cleaning | 27 / 30 |
| B. Exploratory Data Analysis | 19 / 25 |
| C. Predictive Modelling | 26 / 30 |
| D. Model Improvement | 22 / 30 |
| **Subtotal (Notebook)** | **94 / 115** |
