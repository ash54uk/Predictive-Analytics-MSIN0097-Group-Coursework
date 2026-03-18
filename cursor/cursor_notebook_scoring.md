# Cursor — Main Notebook Scoring

**Notebook:** `manchester_property_analysis.ipynb`
**First-attempt success:** Yes
**Iterations needed:** 1
**Runs end-to-end without errors:** Yes

---

## Part A: Data Ingestion & Cleaning (22/30)

| Criterion | Score | Justification |
|---|---|---|
| **Schema exploration** | 4/5 | Cell-3: prints shape, columns, missing counts. Cell-6: checks unique values per column, identifies constants/identifiers. Cell-7: price percentiles (1st, 5th, 95th, 99th). Missing: no % missing summary table. |
| **Uninformative column detection** | 5/5 | Cell-6 explicitly checks for constant, unique-per-row, and low-unique columns. Cell-8 decision table correctly identifies `nearest_school_ofsted` as useless despite zero nulls. |
| **Missingness handling** | 3/5 | Per-column justification in cell-8 decision table. Preserves `floor_area` via median imputation by property_type (good). But drops `epc_rating` entirely (a strong predictor). Does NOT notice that epc_rating, floor_area, and co2_emissions are missing for the exact same rows (co-missingness pattern). |
| **Outlier handling** | 3/5 | Cell-7 checks price stats and extreme value counts. Cell-9 removes <£10k and >£2M with justification. Does not investigate specific extreme values (£1 transactions, £292M) individually — applies a threshold only. |
| **Date handling** | 3/5 | Cell-9: converts to datetime, extracts year and month. No quarter, no seasonal analysis, no inflation adjustment considered. |
| **Documentation** | 4/5 | Cell-8 has a clear decision table with justification for each cleaning choice. Markdown cells throughout. Some justifications are terse. |

### Key Differentiators

- Detected `nearest_school_ofsted` as useless despite zero nulls
- Preserved `floor_area` but dropped `epc_rating` (missed opportunity)
- Did not flag specific extreme prices (£1, £292M)
- Did not notice the co-missingness pattern across EPC-related columns

---

## Part B: Exploratory Data Analysis (15/25)

| Criterion | Score | Justification |
|---|---|---|
| **Plot variety** | 3/5 | 6 plots across 4 cells: price histogram (raw + log), boxplot by property type, bar chart by borough, temporal trend (median + volume), correlation heatmap. No spatial map (lat/lon). |
| **Plot quality** | 4/5 | Titles, labels, appropriate sizing. Clean formatting using seaborn + matplotlib. Not quite publication-ready but solid. |
| **Insight depth** | 3/5 | Some specific observations: "Trafford and Stockport show higher median prices", "notable dip around 2020 (COVID-19)". Mostly descriptive, not deeply quantified. |
| **Feature relationships** | 3/5 | Bivariate: price x property type, price x borough, price over time. Correlation matrix. No interaction exploration (e.g. price x borough x property type). |
| **Handling of own cleaning choices** | 2/5 | Does not address gaps from dropping epc_rating or co2_emissions. Does not acknowledge limitations from cleaning decisions. |

### Key Differentiators

- No spatial map (lat/lon coloured by price)
- Does explore temporal trends
- Insights are descriptive but not deeply quantified
- Does not consider impact of own cleaning decisions on EDA scope

---

## Part C: Predictive Modelling (23/30)

| Criterion | Score | Justification |
|---|---|---|
| **Model choice** | 4/5 | Cell-24: "Random Forest — handles mixed feature types, non-linearity, and requires minimal preprocessing. Robust to outliers." Good justification. |
| **Feature selection** | 4/5 | Drops identifiers, high-cardinality columns (postcode, name columns). Keeps useful numeric + categorical features. |
| **Encoding strategy** | 3/5 | OneHotEncoder (drop='first', handle_unknown='ignore') applied uniformly to all categoricals via ColumnTransformer. No ordinal encoding for ordered features. |
| **Data leakage prevention** | 5/5 | Uses sklearn Pipeline with ColumnTransformer. Split happens before fit. StandardScaler and OneHotEncoder are inside the pipeline, so they are only fit on training data during `model_baseline.fit(X_train, y_train)`. |
| **Evaluation** | 4/5 | RMSE, MAE, R² reported. Predicted vs actual scatter + residual histogram + feature importance chart. Good visual diagnostics. |
| **Interpretation** | 3/5 | Cell-28: "Floor area, location, and property type are the strongest predictors." Brief. Does not explain why R²=0.63 or identify model weaknesses. |

### Actual Baseline Results

| Metric | Value |
|---|---|
| RMSE | £96,597 |
| MAE | £50,167 |
| R² | 0.6330 |

---

## Part D: Model Improvement (17/30)

| Criterion | Score | Justification |
|---|---|---|
| **Strategy variety** | 4/5 | Four strategies: log-transform target, switch to GradientBoosting, add town feature, add floor_area_ratio. |
| **Feature engineering** | 3/5 | floor_area_ratio (floor_area / median by property_type) is creative. Added town. Log10 target transform. Limited scope overall. |
| **Improvement magnitude** | 1/5 | The "improved" model performed worse. RMSE went UP from £96,597 to £102,048 (+5.6%). R² dropped from 0.6330 to 0.5904. Only MAE slightly improved (-3.4%). |
| **Fair comparison** | 4/5 | Same random_state=42, same test_size=0.2. Converts back from log space. Side-by-side metrics table in cell-35 with % change. |
| **Leakage awareness** | 3/5 | No target leakage. But `floor_area_ratio` is computed using groupby on the full dataset before the split — mild data leakage from test set statistics. Not acknowledged. |
| **Honesty** | 2/5 | Cell-36 hedges with "Results may vary... If the improved model outperforms the baseline, these factors are likely contributors." This is evasive — the model clearly performed worse but the text does not own that result. Reads like pre-written commentary not updated after seeing actual output. |

### Actual Improved Results

| Metric | Baseline (RF) | Improved (GB) | Change |
|---|---|---|---|
| RMSE | £96,597 | £102,048 | +5.6% |
| MAE | £50,167 | £48,453 | -3.4% |
| R² | 0.6330 | 0.5904 | -0.0426 |

---

## Overall Summary

| Section | Score | Max |
|---|---|---|
| A. Data Ingestion & Cleaning | 22 | 30 |
| B. Exploratory Data Analysis | 15 | 25 |
| C. Predictive Modelling | 23 | 30 |
| D. Model Improvement | 17 | 30 |
| **TOTAL (Parts A–D)** | **77** | **115** |

### Notable Failures and Surprises

- The "improved" model performed **worse** than the baseline (R² dropped from 0.63 to 0.59), and the commentary did not acknowledge this
- No spatial visualisation despite having lat/lon data
- Dropped `epc_rating` entirely without exploring imputation or ordinal encoding
- Did not notice the co-missingness pattern across EPC-related columns
