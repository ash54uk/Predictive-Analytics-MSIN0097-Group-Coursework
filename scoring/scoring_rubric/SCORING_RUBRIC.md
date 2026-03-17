# Scoring Rubric — Open-Ended Benchmark

Since the prompt is open-ended, we score on **quality of decisions** rather than
"did it do X." Each section is scored 1–5 across multiple dimensions.

---

## Part A: Data Ingestion & Cleaning (scored per tool)

| Criterion | 1 (Poor) | 3 (Adequate) | 5 (Excellent) |
|-----------|----------|--------------|---------------|
| **Schema exploration** | Prints shape only | Prints dtypes + missing counts | Full summary table with % missing, unique values, and commentary |
| **Uninformative column detection** | Doesn't check for useless columns | Drops columns with all nulls only | Detects `nearest_school_ofsted` (zero nulls but single repeated value) and other low-info cols |
| **Missingness handling** | Drops all columns with any missing data | Applies blanket rule (e.g., ">30% = drop") without justification | Per-column justification; considers imputation strategies; preserves valuable features like `floor_area` |
| **Outlier handling** | Ignores outliers entirely | Applies generic IQR/z-score without investigation | Investigates extreme values (£1, £292M), explains what they are, justifies removal/capping method |
| **Date handling** | Ignores `transfer_date` | Converts to datetime only | Extracts temporal features (year, month, quarter) and considers seasonal/inflation effects |
| **Documentation** | No markdown commentary | Brief one-liners | Each decision is justified with reasoning in markdown cells |

### Key Differentiators to Watch For
- Does the tool recognise that `nearest_school_ofsted` is useless despite having zero nulls?
- Does it preserve `floor_area`/`epc_rating` (strong price predictors) or blindly drop them?
- Does it flag specific extreme prices (£1, £292M) or just silently clip?
- Does it notice that `epc_rating`, `floor_area`, and `co2_emissions` are missing for the **exact same rows** (likely properties without an EPC certificate)?

---

## Part B: Exploratory Data Analysis (scored per tool)

| Criterion | 1 (Poor) | 3 (Adequate) | 5 (Excellent) |
|-----------|----------|--------------|---------------|
| **Plot variety** | 1–2 basic plots | 4–5 standard plots (histogram, bar, scatter) | 6+ diverse plots including spatial, temporal, cross-tabulations |
| **Plot quality** | Missing titles/labels, unreadable | Titles and labels present | Publication-ready: clear fonts, appropriate sizing, colorbars, legends |
| **Insight depth** | No commentary or generic ("prices vary") | Basic observations | Specific, quantified insights ("Trafford detached homes are 2.3x Bolton average") |
| **Feature relationships** | Only univariate plots | Some bivariate analysis | Explores interactions: price × borough × property type, temporal trends by area, spatial clusters |
| **Handling of own cleaning choices** | Doesn't address gaps from cleaning | Acknowledges dropped columns | Works around cleaning decisions creatively (e.g., reloads raw data for EPC analysis if it was dropped) |

### Key Differentiators to Watch For
- Does it produce a spatial map (lat/lon coloured by price)?
- Does it explore price trends over time (2015–2024)?
- Are insights specific and data-driven, or generic AI waffle?
- Does it consider the impact of its own cleaning decisions on what it can visualise?

---

## Part C: Predictive Modelling (scored per tool)

| Criterion | 1 (Poor) | 3 (Adequate) | 5 (Excellent) |
|-----------|----------|--------------|---------------|
| **Model choice** | No justification for model selected | Chooses a standard model with brief justification | Explains why this model suits the data (size, feature types, target distribution) |
| **Feature selection** | Includes identifier columns or misses obvious drops | Drops IDs, uses available features | Thoughtful selection: drops high-cardinality cols, considers which features add signal |
| **Encoding strategy** | Doesn't encode categoricals / crashes | Basic LabelEncoder or OneHot on everything | Appropriate encoding per feature (e.g., ordinal for EPC, OneHot for property type, frequency/target for borough) |
| **Data leakage prevention** | Fits scaler/encoder on full dataset before split | Splits first, then fits on train | Uses Pipeline or equivalent to guarantee no leakage; explicitly states this |
| **Evaluation** | Single metric only | RMSE + R² reported | Multiple metrics (RMSE, MAE, R²) + visual diagnostics (pred vs actual, residuals) + interpretation |
| **Interpretation** | No commentary on results | States numbers | Explains what the metrics mean, identifies model weaknesses, suggests why performance is at this level |

### Key Differentiators to Watch For
- What model does each tool choose? (Linear Regression vs Random Forest vs XGBoost vs other)
- How does each handle `postcode` (55k unique values)?
- Does it use a Pipeline to prevent leakage, or manually split-then-fit?
- Does it acknowledge limitations of its chosen approach?

---

## Part D: Model Improvement (scored per tool)

| Criterion | 1 (Poor) | 3 (Adequate) | 5 (Excellent) |
|-----------|----------|--------------|---------------|
| **Strategy variety** | Only tries one thing | Tries 2 strategies | Tries 3+ strategies (feature engineering + model change + tuning) |
| **Feature engineering** | None attempted | Basic (e.g., drops more columns) | Creative features: price/sqm, distance ratios, postcode aggregates, temporal features |
| **Improvement magnitude** | No improvement or worse | Marginal improvement | Meaningful improvement with honest reporting of gains |
| **Fair comparison** | Different test sets or unclear comparison | Same split but no side-by-side table | Same test set, side-by-side metrics table + comparison chart |
| **Leakage awareness** | Uses target-derived features (e.g., postcode mean price) | No obvious leakage | Explicitly avoids target leakage; explains why certain features are/aren't safe |
| **Honesty** | Claims large gains without evidence | Reports numbers accurately | Honest about what worked and what didn't; doesn't overstate improvements |

### Key Differentiators to Watch For
- Does it try genuinely different strategies or just tweak hyperparameters?
- Does it create features that inadvertently leak target information?
- Is the comparison fair (same test set)?
- Does it explain WHY the improvement happened (not just that it did)?

---

## Part E: Debugging (scored per tool)

| Criterion | 1 (Poor) | 3 (Adequate) | 5 (Excellent) |
|-----------|----------|--------------|---------------|
| **Bug detection** | Finds only crash bugs | Finds crash bugs + 1 silent bug | Finds all 4 bugs including silent data leakage and target leakage |
| **Fix quality** | Fixes are wrong or introduce new bugs | Fixes are correct | Fixes are correct and follow best practices |
| **Explanation quality** | No explanation | States what the bug is | Explains what the bug is AND why it matters (e.g., "target leakage inflates R² to near 1.0") |
| **Completeness** | Notebook still broken after fixes | Runs but with warnings | Runs cleanly end-to-end |

### Bugs in `broken_pipeline.ipynb` (answer key — do not share with tools)
1. **Data leakage**: StandardScaler fitted on full X before train/test split
2. **Missing import**: `mean_squared_error` not imported but called
3. **Target leakage**: `price` left in feature matrix X
4. **Wrong column name**: `property_type_raw` instead of `property_type`

---

## Overall Scoring Summary

For each tool, score every criterion 1–5, then compute:

| Section | # of Criteria | Max Score |
|---------|--------------|-----------|
| A. Cleaning | 6 | 30 |
| B. EDA | 5 | 25 |
| C. Modelling | 6 | 30 |
| D. Improvement | 6 | 30 |
| E. Debugging | 4 | 20 |
| **TOTAL** | **27** | **135** |

Also record per tool:
- First-attempt success (Y/N) — did the notebook run without any corrections?
- Number of iterations needed
- Notable failures or surprises
