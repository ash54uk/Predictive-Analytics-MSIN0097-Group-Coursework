# Predictive-Analytics-MSIN0097-Group-Coursework
# Manchester Property Price Prediction — AI Coding Agent Benchmark

Benchmarking three AI coding agents (Claude Code, Cursor, Antigravity) on an end-to-end data science pipeline for predicting residential property sale prices across Greater Manchester (2015–2024).

## Project Overview

Each AI agent was given the same prompt and dataset and asked to:
1. Build a complete data science notebook (ingestion, EDA, modelling, improvement)
2. Debug a separate broken ML pipeline (`broken_pipeline.ipynb`)

The outputs were scored on code quality, analytical depth, model performance, and debugging accuracy. Results are compiled in `misc/Agent_Benchmarking_Framework.xlsx`.

## Project Structure

```
.
├── README.md
├── misc/
│   ├── PROMPT.md                          # Exact prompt given to all three agents
│   └── Agent_Benchmarking_Framework.xlsx  # Scoring rubric and results
├── claude/
│   ├── manchester_property_analysis.ipynb # Claude Code — main notebook
│   ├── broken_pipeline.ipynb              # Claude Code — debugging task
│   └── requirements.txt
├── cursor/
│   ├── manchester_property_analysis.ipynb # Cursor — main notebook
│   ├── broken_pipeline.ipynb              # Cursor — debugging task
│   ├── requirements.txt
│   ├── INTERACTION_LOG.md                 # Cursor interaction log
│   ├── cursor_notebook_scoring.md         # Cursor notebook scoring
│   └── cursor_debugging_scoring.md        # Cursor debugging scoring
└── antigravity/
    ├── manchester_house_prices.ipynb       # Antigravity — main notebook
    ├── broken_pipeline.ipynb              # Antigravity — debugging task
    ├── requirements.txt
    ├── Interaction Log Antigravity.md     # Antigravity interaction log
    ├── antigravity_notebook_scoring.md    # Antigravity notebook scoring
    └── antigravity_debugging_scoring.md   # Antigravity debugging scoring
```

## Dataset

The dataset (`manchester_features.parquet`) is a Spark-partitioned parquet directory containing 474,144 property transactions with 23 columns. **Note:** the dataset is not included in this repository due to size.

| Feature | Description |
|---|---|
| `price` | Sale price (GBP) — target variable |
| `property_type` | D=Detached, S=Semi-detached, T=Terraced, F=Flat, O=Other |
| `old_new` | Y=new build, N=established |
| `duration` | F=Freehold, L=Leasehold |
| `borough` | One of 10 Greater Manchester boroughs |
| `epc_rating` | Energy efficiency rating (A–G); missing for ~33% of rows |
| `floor_area` | Floor area in m² (EPC-linked, ~33% missing) |
| `co2_emissions` | CO₂ emissions (EPC-linked, ~33% missing) |
| `lat` / `lon` | Coordinates |
| `crime_count` | Crime count in the surrounding area |
| `dist_nearest_station_km` | Distance to nearest rail/metro station |
| `dist_nearest_school_km` | Distance to nearest school |
| `dist_nearest_supermarket_km` | Distance to nearest supermarket |
| `transfer_date` | Transaction date (2015–2024) |

## Setup

Each agent subdirectory has its own `requirements.txt`. To run a specific agent's notebooks:

```bash
cd claude   # or cursor / antigravity
pip install -r requirements.txt
```

> **Note:** When loading the dataset, use `engine='fastparquet'` — pyarrow triggers a histogram bug on Spark-partitioned parquet files.

```python
df = pd.read_parquet('manchester_features.parquet/manchester_features.parquet', engine='fastparquet')
```

## Bugs Found in `broken_pipeline.ipynb`

All three agents were asked to find and fix bugs in the same broken pipeline. The four known bugs are:

| # | Bug | Impact |
|---|---|---|
| 1 | `StandardScaler` fitted on the full dataset before `train_test_split` | Data leakage — test-set statistics contaminate training, inflating evaluation metrics |
| 2 | `mean_squared_error` used but never imported | `NameError` crash at evaluation; no metrics produced |
| 3 | `price` column included in feature matrix `X` | Target leakage — model trivially predicts price from price |
| 4 | Categorical encoding referenced `property_type_raw` (non-existent column) | `KeyError` crash; pipeline never reaches training |

## Module: MSIN0097 Predictive Analytics — UCL School of Management
