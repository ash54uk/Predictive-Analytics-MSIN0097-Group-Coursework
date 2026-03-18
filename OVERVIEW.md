# Project Overview — MSIN0097 Predictive Analytics Group Coursework

## What This Project Is

This is a benchmarking study that compares three AI coding assistants — **Claude Code**, **Cursor**, and **Antigravity** — on their ability to independently produce a complete, end-to-end data science pipeline from a single natural language prompt.

Each tool received exactly the same prompt and the same dataset. The outputs are then scored against a shared rubric to evaluate the quality of each tool's work across data cleaning, EDA, modelling, model improvement, and debugging.

---

## The Task

Each tool was asked to:

1. **Build a full ML notebook** — load and clean the Manchester property dataset, perform EDA, build a baseline predictive model, then improve it
2. **Debug a broken pipeline** — identify and fix 4 bugs (two silent data leakage issues, one crash from a missing import, one crash from a wrong column name)

The prompt is in [`scoring/PROMPT.md`](scoring/PROMPT.md). The evaluation criteria are in [`scoring/scoring_rubric/SCORING_RUBRIC.md`](scoring/scoring_rubric/SCORING_RUBRIC.md).

---

## The Dataset

**`manchester_features.parquet`** — 474,144 residential property transactions across Greater Manchester (2015–2024), stored as a Spark-partitioned Parquet directory.

Key features:

| Feature | Description |
|---|---|
| `price` | Sale price in GBP — the target variable |
| `property_type` | D=Detached, S=Semi-detached, T=Terraced, F=Flat, O=Other |
| `old_new` | Y=new build, N=established |
| `duration` | F=Freehold, L=Leasehold |
| `borough` | One of 10 Greater Manchester boroughs |
| `epc_rating` | Energy efficiency (A–G); ~33% missing |
| `floor_area` | Floor area in m²; ~33% missing (co-missing with EPC) |
| `co2_emissions` | CO₂ emissions; ~33% missing (co-missing with EPC) |
| `lat` / `lon` | Property coordinates |
| `crime_count` | Crime count in surrounding area |
| `dist_nearest_station_km` | Distance to nearest rail/metro station |
| `dist_nearest_school_km` | Distance to nearest school |
| `dist_nearest_supermarket_km` | Distance to nearest supermarket |
| `transfer_date` | Date of sale |

> **Note:** Read with `engine='fastparquet'` — pyarrow triggers a histogram bug on Spark-partitioned files.

---

## Repository Structure

```
.
├── README.md                        # Dataset description and bug list
├── OVERVIEW.md                      # This file
├── manchester_features.parquet/     # Shared dataset (all tools use this)
│
├── claude/                          # Claude Code output
│   ├── manchester_property_analysis.ipynb   # Main pipeline notebook
│   ├── broken_pipeline.ipynb                # Debugging task (fixed)
│   └── requirements.txt
│
├── cursor/                          # Cursor output
│   ├── manchester_property_analysis.ipynb
│   ├── broken_pipeline.ipynb
│   ├── cursor_notebook_scoring.md           # Scored: 77/115
│   ├── cursor_debugging_scoring.md          # Scored: 15/20
│   └── requirements.txt
│
├── antigravity/                     # Antigravity output
│   ├── manchester_house_prices.ipynb
│   ├── broken_pipeline.ipynb
│   └── requirements.txt
│
└── scoring/
    ├── PROMPT.md                    # Exact prompt given to all 3 tools
    └── scoring_rubric/
        └── SCORING_RUBRIC.md        # 5-part rubric, 135 points total
```

---

## What Claude Did

### Main Notebook — `claude/manchester_property_analysis.ipynb`

Claude produced the strongest overall pipeline, achieving the highest R² of the three tools.

#### 1. Data Ingestion & Cleaning
- Loaded the Spark-partitioned Parquet file with `engine='fastparquet'`
- Dropped uninformative columns: IDs (`tx_id`), text names (`nearest_station_name`, etc.), and `nearest_school_ofsted` (low variance)
- Parsed `transfer_date` to extract `year` and `month`
- Removed price outliers using the 1st–99th percentile range
- Capped extreme `floor_area` values
- Added a `has_epc` binary indicator to flag rows where EPC data (rating, floor area, CO₂) is missing — preserving missingness as a signal rather than discarding it

#### 2. Exploratory Data Analysis
- Price distribution histograms showing the right skew and justifying log-transformation of the target
- Median price by borough — identifying Trafford and Stockport as premium areas
- Price by property type — Detached > Semi > Terraced > Flat
- New-build premium analysis — approximately 25–30% uplift over established properties
- Price trends 2015–2024 — upward trajectory with a COVID-19 dip visible in 2020
- Floor area vs price scatter plot; EPC rating vs price box plots
- Correlation heatmap across all numeric features

#### 3. Baseline Model — Ridge Regression
- Log-transformed the target (`price`) to handle right skew
- Used an sklearn `Pipeline` with `ColumnTransformer` to prevent data leakage (scaling and encoding happen inside the pipeline, after the train/test split)
- One-hot encoded categorical features; standardised numeric features
- **Baseline results: R² ≈ 0.52, MdAPE ≈ 22.8%**

#### 4. Model Improvement — Random Forest + XGBoost
Feature engineering additions:
- **Cyclical month encoding** — sine and cosine transforms of month to capture seasonal patterns without ordinal gaps (December→January continuity)
- **Total proximity score** — aggregate of the three distance features into a single composite score

Two improved models were trained: Random Forest and XGBoost (with `random_state=42` throughout).

**Best improved results: R² ≈ 0.75, MdAPE ≈ 13.7%** — a significant improvement over baseline.

Evaluation outputs include actual vs predicted scatter plots, residual histograms, and feature importance bar charts showing which features drove predictions most.

---

### Debugging Notebook — `claude/broken_pipeline.ipynb`

The original `broken_pipeline.ipynb` contained 4 bugs. Claude identified and fixed all of them:

| # | Bug | Type | Fix |
|---|---|---|---|
| 1 | `StandardScaler` fitted on the full dataset **before** `train_test_split` | Data leakage | Move the split before any fitting; fit scaler only on training data |
| 2 | `mean_squared_error` used but not imported | Missing import (crash) | Add `from sklearn.metrics import mean_squared_error` |
| 3 | `price` column included in feature matrix `X` | Target leakage | Drop `price` from `X` before training |
| 4 | Encoding referenced `property_type_raw` (column does not exist) | Wrong column name (crash) | Change to `property_type` |

For each bug, Claude explained what the bug was, why it matters (data leakage vs crash vs silent wrong results), and showed the corrected code.

---

## How the Three Tools Compare

| Metric | Claude | Cursor | Antigravity |
|---|---|---|---|
| Baseline model | Ridge (R² ≈ 0.52) | Random Forest (R² ≈ 0.63) | Ridge (R² ≈ 0.44) |
| Improved model | RF + XGBoost (R² ≈ 0.75) | GradientBoosting (R² ≈ 0.59*) | GradientBoosting (R² ≈ 0.51) |
| Data leakage prevention | sklearn Pipeline | sklearn Pipeline | Manual split |
| Feature engineering | Cyclical encoding, proximity score | Minimal | Log-transform target |
| Debugging (bugs found) | 4/4 | 4/4 | — |

*Cursor's "improved" model actually performed worse than its baseline.

---

## Scoring Rubric Summary

The notebooks are evaluated across 5 sections (135 points total):

| Section | Criteria | Max Points |
|---|---|---|
| A: Data Ingestion & Cleaning | Quality of cleaning decisions, justification, leakage avoidance | 30 |
| B: Exploratory Data Analysis | Visualisation quality, insights, commentary | 25 |
| C: Predictive Modelling | Model choice, encoding, evaluation, interpretation | 30 |
| D: Model Improvement | Strategy, feature engineering, quantitative comparison | 30 |
| E: Debugging | Bug identification, explanation of impact, correct fix | 20 |

Cursor's scored cards are included in `cursor/cursor_notebook_scoring.md` (77/115) and `cursor/cursor_debugging_scoring.md` (15/20) as reference examples.
