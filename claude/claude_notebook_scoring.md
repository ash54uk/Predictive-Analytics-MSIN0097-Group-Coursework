
# Claude Code — Main Notebook Scoring

**Tool:** Claude Code (VS Code extension)  
**Task:** End-to-end data science notebook from shared prompt  
**Output:** `manchester_property_analysis.ipynb` (80 cells: 35 code, 45 markdown)  
**First-attempt success:** No  
**Total interactions:** 3 (notebook generation, parquet loading debug, savefig fix)  

---

## Execution Summary

| # | User Message | Claude Code Response | Resolved |
|---|-------------|---------------------|----------|
| 1 | Standard benchmark prompt — produce end-to-end notebook | Explored dataset from command line (schema, dtypes, missing values, cardinalities, price distribution), then generated complete notebook | Y |
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

**Dataset:** 474,144 transactions | **Cleaned rows:** 464,689

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

## Part B: Exploratory Data Analysis (22/25)

| Criterion | Score | Justification |
|-----------|-------|---------------|
| Plot variety | 5 | 10 plots across diverse types: price distribution (raw + log), borough median bar, property type box, quarterly price trend, floor area scatter, EPC rating bar, new-build premium comparison, correlation heatmap, transaction volume by year, **and spatial price map** (lat/lon coloured by price, 50k sample). |
| Plot quality | 4 | Formatted axes with £k labels, titles, colourbars, RdYlGn colourmap for spatial map, appropriate sizing throughout. Publication-ready quality. |
| Insight depth | 4 | Specific quantified insights: Trafford/Stockport premium 50–70% above Oldham/Rochdale, new-build premium ~25–30%, COVID volume dip 2020, stamp duty surge 2021, floor area Pearson r ≈ 0.55, counterintuitive EPC-A price finding explained by composition effect. |
| Feature relationships | 4 | Bivariate analysis throughout: floor area vs price, EPC rating vs price, property type vs price, borough vs price, new-build vs established. Correlation matrix covers all numeric features. Spatial map validates lat/lon as predictors. |
| Handling of cleaning choices | 5 | Used `has_epc` filter for EPC-specific plots. Spatial map (Section 2.10) uses lat/lon coloured by price with markdown insight explaining the geographic price structure. Validates modelling decisions directly. |

> **Strengths:** The spatial map (Section 2.10) is a key differentiator the rubric specifically rewards — Claude Code included it with a 50k sample coloured by log price, accompanied by a detailed insight about southern vs northern borough price gradients.

---

## Part C: Predictive Modelling (26/30)

| Criterion | Score | Justification |
|-----------|-------|---------------|
| Model choice | 5 | Ridge regression as baseline on log-transformed price (`log1p`), with clear justification: linear, interpretable, establishes performance floor. Log transform explained as reducing skew and penalising proportional errors. |
| Feature selection | 4 | Dropped all IDs and high-cardinality columns. Kept distance features, coordinates, temporal features. 80/20 split stratified by borough. |
| Encoding strategy | 5 | Ordinal encoding for EPC rating (A→G natural order). OneHot for property type, old/new, duration, borough. ColumnTransformer with separate pipelines for numeric (median impute + scale), OHE, and ordinal features. |
| Data leakage prevention | 5 | sklearn Pipeline with ColumnTransformer used throughout — imputation, scaling, and encoding fitted only on training data. Train/test split done before any transformation. |
| Evaluation | 4 | MAE, RMSE, R², and MdAPE reported. Actual vs predicted scatter plot with perfect-prediction line. |
| Interpretation | 3 | Explained metrics and identified underprediction at high prices. Could go deeper on why Ridge specifically struggles (R² = 0.52) and what nonlinear interactions are missing. |

---

## Part D: Model Improvement (27/30)

| Criterion | Score | Justification |
|-----------|-------|---------------|
| Strategy variety | 4 | Three distinct strategies: extensive feature engineering + Random Forest + XGBoost with tuned hyperparameters. |
| Feature engineering | 5 | Six engineered features: (1) cyclical month sin/cos, (2) total proximity score, (3) floor area band indicator, (4) borough median target encoding from training data only, (5) property type median target encoding from training data only, (6) borough × property_type interaction term. |
| Improvement magnitude | 4 | R² improved from 0.52 (Ridge) to 0.75 (Random Forest/XGBoost). MAE dropped from £60k to £40k. Substantial improvement. |
| Fair comparison | 5 | Same test set used throughout. Side-by-side metrics table (MAE, RMSE, R², MdAPE) and comparison bar chart. Three models compared quantitatively. |
| Leakage awareness | 5 | Explicitly computed `borough_median_price` and `type_median_price` from training data only with comment "no leakage". Target encoding derived from `X_train.join(y_train)` — not full dataset. |
| Honesty | 4 | Honest reporting — noted RF and XGBoost performed similarly (both R² = 0.75). Didn't overstate XGBoost gains. Suggested further improvements (Optuna tuning, spatial features, stacking). |

> **Strengths:** Feature engineering was significantly richer than a basic attempt. Borough × property_type interaction captures location-specific premiums. Target encoding explicitly derived from training data only, demonstrating clear leakage awareness.

---

## Overall Score

| Section | Score |
|---------|-------|
| A. Data Ingestion & Cleaning | 27 / 30 |
| B. Exploratory Data Analysis | 22 / 25 |
| C. Predictive Modelling | 26 / 30 |
| D. Model Improvement | 27 / 30 |
| **Subtotal (Notebook)** | **102 / 115** |
