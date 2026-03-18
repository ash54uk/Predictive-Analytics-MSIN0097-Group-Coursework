# Predictive-Analytics-MSIN0097-Group-Coursework
# Manchester Property Price Prediction

End-to-end data science pipeline for predicting residential property sale prices across Greater Manchester (2015–2024), covering 474,144 transactions.

## Project structure

```
.
├── manchester_features.parquet/   # Spark-partitioned dataset (474k transactions, 23 columns)
├── manchester_property_analysis.ipynb  # Full EDA + modelling notebook
├── broken_pipeline.ipynb          # Standalone pipeline (simplified, fixed)
├── requirements.txt
└── README.md
```

## Dataset

`manchester_features.parquet` is a Spark-partitioned directory. Each row is a property transaction with features including:

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

```bash
pip install -r requirements.txt
```

> **Note:** `manchester_features.parquet` is a Spark-partitioned directory. Use `engine='fastparquet'` when reading with pandas — pyarrow triggers a histogram bug on these files.

```python
df = pd.read_parquet('manchester_features.parquet/manchester_features.parquet', engine='fastparquet')
```

## Notebooks

### `manchester_property_analysis.ipynb` — full pipeline

Covers the complete workflow:

1. **Data ingestion & cleaning** — drops uninformative columns, parses dates, removes price outliers (1st–99th percentile), caps floor area extremes, adds `has_epc` missingness indicator
2. **EDA** — price distributions, borough medians, property type breakdown, price trends (2015–2024), floor area vs price, EPC rating vs price, new-build premium (~25–30%), correlation matrix
3. **Baseline model** — Ridge regression on log-transformed price; R² ≈ 0.52, MdAPE ≈ 22.8%
4. **Improved models** — Random Forest and XGBoost with feature engineering (cyclical month encoding, total proximity score); best R² ≈ 0.75, MdAPE ≈ 13.7%
5. **Evaluation** — actual vs predicted plots, residual analysis, feature importances

**Model results summary:**

| Model | MAE | RMSE | R² | MdAPE |
|---|---|---|---|---|
| Ridge (baseline) | £60,340 | £95,776 | 0.52 | 22.8% |
| Random Forest | £40,237 | £70,684 | 0.75 | 13.7% |
| XGBoost | £41,237 | £71,272 | 0.75 | 14.4% |

### `broken_pipeline.ipynb` — standalone simplified pipeline

A self-contained Random Forest pipeline. Fixed bugs (see below) allow it to run end-to-end.

## Bugs fixed in `broken_pipeline.ipynb`

| # | Bug | Impact |
|---|---|---|
| 1 | `StandardScaler` fitted on the full dataset before `train_test_split` | Data leakage — test-set statistics contaminate training, inflating evaluation metrics |
| 2 | `mean_squared_error` used but never imported | `NameError` crash at evaluation; no metrics produced |
| 3 | `price` column included in feature matrix `X` | Target leakage — model trivially learns to predict price from price, producing meaningless near-perfect R² |
| 4 | Categorical encoding referenced `property_type_raw` (non-existent column) | `KeyError` crash; pipeline never reaches training |

## Key findings

- **Location dominates:** `lat`/`lon` and `borough` are the top predictors. Trafford and Stockport command median prices 50–70% above Oldham and Rochdale.
- **Floor area** is the strongest property-level predictor (Pearson r ≈ 0.55 in the EPC-linked sample).
- **New-build premium:** ~25–30% over equivalent established properties.
- **Price cycle:** Steady growth 2015–2022, post-COVID surge in 2021, correction in 2023–2024 as interest rates rose.
- **EPC missingness is structural:** ~33% of properties have never had an EPC filed. A `has_epc` binary indicator captures this signal rather than silently imputing.
