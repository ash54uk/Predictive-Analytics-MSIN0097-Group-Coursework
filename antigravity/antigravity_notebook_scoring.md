# Antigravity — Main Notebook Scoring

**Notebook:** `manchester_house_prices.ipynb`
**First-attempt success:** Yes (V2 single notebook)
**Iterations needed:** 1
**Runs end-to-end without errors:** Yes (but Gradient Boosting took 50+ minutes)

---

## Part A: Data Ingestion & Cleaning (16/30)

| Criterion | Score | Justification |
|---|---|---|
| **Schema exploration** | 2/5 | Cell-3: prints shape (474144, 23) and missing counts for columns with >0 missing. Does NOT print dtypes, unique values, or a full summary table for all 23 columns. |
| **Uninformative column detection** | 2/5 | Cell-9: drops categorical columns with >1000 unique values. Cell-4: drops columns with >50% missing. Does NOT detect `nearest_school_ofsted` (zero nulls, near-constant value) — kept it as a categorical feature and OneHotEncoded it. |
| **Missingness handling** | 2/5 | Blanket rule: >50% = drop, else median (numeric) / mode (categorical). No per-column justification. Did NOT notice that `epc_rating`, `floor_area`, and `co2_emissions` are missing for the exact same rows (co-missingness pattern). |
| **Outlier handling** | 3/5 | Cell-4: removes price ≤ £1,000 and caps at 99th percentile. Reasonable method but never investigates or flags specific extreme values (£1 transactions, £292M mansions) — silently removes them. |
| **Date handling** | 4/5 | Cell-4: converts `transfer_date` to datetime, extracts `sale_year`, `sale_month`, and `sale_age`. Good temporal features. But `sale_age = 2024 - sale_year` is hardcoded — breaks future reproducibility. |
| **Documentation** | 3/5 | Cell-2 markdown has structured cleaning rationale, but written as a generic template before seeing the data (e.g., "like 'county' if all are Greater Manchester" as a hypothetical example). Not driven by actual findings. |

### Key Differentiators

- Did NOT detect `nearest_school_ofsted` as useless — kept it as a feature
- Applied blanket missingness rules without per-column justification
- Did not notice the co-missingness pattern across EPC-related columns
- Created `sale_age` feature (unique to Antigravity) but hardcoded the reference year
- Documentation is pre-written template, not data-driven

---

## Part B: Exploratory Data Analysis (12/25)

| Criterion | Score | Justification |
|---|---|---|
| **Plot variety** | 2/5 | Only 2 plots: price distribution (raw + log-transformed histogram with KDE) and average price over time (line chart with markers). No box plots by property type, no correlation heatmap, no spatial map, no borough analysis. |
| **Plot quality** | 4/5 | Both plots have titles, axis labels, appropriate sizing (figsize specified). KDE overlay on histogram, markers on line plot. Clean formatting. Not publication-ready but solid. |
| **Insight depth** | 2/5 | Two generic print statements: "heavily skewed to the right" and "highlighting periods of inflation or market shifts." No specific numbers, no quantified comparisons, no actionable observations. |
| **Feature relationships** | 1/5 | Purely univariate analysis. Never explores price × property type, price × borough, price × distance features, or any bivariate/interaction relationships. |
| **Handling of own cleaning choices** | 3/5 | Did not drop EPC columns (below 50% threshold) so the issue of missing EDA scope didn't arise. Neither good nor bad — neutral. |

### Key Differentiators

- Fewest plots of all tools — only 2 visualisations
- No spatial map despite having lat/lon data
- No categorical breakdowns (property type, borough)
- No correlation heatmap
- Insights are generic print statements, not markdown commentary

---

## Part C: Predictive Modelling (16/30)

| Criterion | Score | Justification |
|---|---|---|
| **Model choice** | 3/5 | Cell-8 markdown: Ridge Regression justified as "robust to multicollinearity." Brief but valid. Does not explain why Ridge suits this specific dataset (474k rows, mixed feature types, skewed target). |
| **Feature selection** | 2/5 | Cell-9: drops strings with >1000 unique values and datetime columns. But keeps `town`, `district`, `nearest_station_name`, `nearest_supermarket_name`, `nearest_school_ofsted` — all high-cardinality or useless columns that inflate the feature matrix. |
| **Encoding strategy** | 1/5 | `OneHotEncoder(handle_unknown='ignore', sparse_output=False)` applied to ALL remaining categoricals including high-cardinality columns (`town`, `district`, `nearest_station_name`, `nearest_supermarket_name`). No ordinal encoding for `epc_rating`. `sparse_output=False` creates a massive dense matrix. Directly caused the 50+ min Gradient Boosting runtime. |
| **Data leakage prevention** | 4/5 | Uses sklearn `Pipeline` with `ColumnTransformer` correctly — `StandardScaler` and `OneHotEncoder` fitted inside the pipeline, guaranteeing train-only fitting. But never explicitly states or explains this benefit. |
| **Evaluation** | 4/5 | Reports R², MAE, RMSE. Reverses log-transform via `expm1()` for interpretable £ metrics. Includes actual vs predicted scatter plot with both models overlaid. |
| **Interpretation** | 2/5 | States the numbers but does not interrogate them. R² = 0.44 is poor for a housing model — never acknowledges this, never explores why, never questions whether the model is fit for purpose. |

### Actual Baseline Results

| Metric | Value |
|---|---|
| RMSE | £98,905 |
| MAE | £59,987 |
| R² | 0.4381 |

---

## Part D: Model Improvement (13/30)

| Criterion | Score | Justification |
|---|---|---|
| **Strategy variety** | 1/5 | Only one strategy: swap Ridge for GradientBoosting. No alternative encoding, no hyperparameter tuning (imported `GridSearchCV` but never used it), no cross-validation, no ensemble methods. |
| **Feature engineering** | 1/5 | Zero new features in the improvement phase. Temporal features (`sale_year`, `sale_month`, `sale_age`) were created during cleaning (scored in Part A). No price/sqm, no distance ratios, no postcode aggregates, no interaction terms attempted to improve the model. |
| **Improvement magnitude** | 2/5 | R² improved from 0.4381 to 0.5066 (+0.0685). MAE improved by only £1,950. Model still explains only ~50% of variance. Marginal gain for 50+ minutes of compute time. |
| **Fair comparison** | 4/5 | Same test set, same preprocessing pipeline, same random_state=42, metrics reported for both models. Comparison scatter plot overlays both models. No side-by-side metrics table though. |
| **Leakage awareness** | 3/5 | No obvious leakage introduced. Pipeline structure maintained throughout. But never explicitly discusses leakage risk or why certain features are/aren't safe to include. |
| **Honesty** | 2/5 | Claims Gradient Boosting "efficiently learns the complex, non-linear geographical hierarchies and price step-functions that are prominent in housing markets" — but improvement was only £1,950 MAE. Does not acknowledge the model is still poor (R² = 0.50) or question whether the approach is working. |

### Actual Improved Results

| Metric | Baseline (Ridge) | Improved (GB) | Change |
|---|---|---|---|
| RMSE | £98,905 | £92,680 | -6.3% |
| MAE | £59,987 | £58,037 | -3.3% |
| R² | 0.4381 | 0.5066 | +0.0685 |

---

## Overall Summary

| Section | Score | Max |
|---|---|---|
| A. Data Ingestion & Cleaning | 16 | 30 |
| B. Exploratory Data Analysis | 12 | 25 |
| C. Predictive Modelling | 16 | 30 |
| D. Model Improvement | 13 | 30 |
| **TOTAL (Parts A–D)** | **57** | **115** |

### Notable Failures and Surprises

- Only 2 EDA plots — fewest of all tools. No box plots, no correlation heatmap, no spatial map
- Did NOT detect `nearest_school_ofsted` as useless — OneHotEncoded it as a feature
- OneHotEncoding high-cardinality columns (`town`, `district`, station/supermarket names) with `sparse_output=False` caused GradientBoosting to run for **50+ minutes**
- Imported `GridSearchCV`, `cross_val_score`, `LinearRegression`, `RandomForestRegressor`, `SimpleImputer`, `os` — **6 unused imports**
- Variable named `rf_pipeline` for GradientBoosting — misleading naming
- `warnings.filterwarnings('ignore')` suppresses all warnings throughout
- Requirements cell lists `fastparquet` but code uses `pyarrow` — inconsistency
- Hardcoded `sale_age = 2024 - sale_year` — breaks in future years
- Zero feature engineering in the improvement phase — only swapped the model
- Did not notice co-missingness pattern across EPC-related columns
- Commentary is generic and pre-written, not updated based on actual results
