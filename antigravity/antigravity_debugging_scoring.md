# Antigravity — Debugging Task Scoring

**Notebook:** `broken_pipeline.ipynb` (fixed version)
**Runs end-to-end after fix:** Yes

---

## Bugs in the Original Notebook (Answer Key)

| # | Bug | Type |
|---|---|---|
| 1 | StandardScaler fitted on full X before train/test split | Data leakage (silent) |
| 2 | `mean_squared_error` not imported but called | Missing import (crash) |
| 3 | `price` left in feature matrix X | Target leakage (silent) |
| 4 | `property_type_raw` instead of `property_type` | Wrong column name (crash) |

---

## What Antigravity Found and Fixed

### Bug 1: Data Leakage — Scaler fit before split (cell-11)

- **Found:** Yes
- **Fixed:** Yes — reordered to split first, then `fit_transform` on train and `transform` on test
- **Explanation provided:** Yes — "Another form of data leakage, sometimes known as temporal or standardisation leakage. By fitting the scaler universally on the entire dataset, statistics from the test set (mean and variance) 'leak' and influence the data given to the model during its training phase."
- **Impact explained:** Yes — "It invalidates the concept of a blind test set, making your test metrics overly optimistic."

### Bug 2: Missing Import — `mean_squared_error` (cell-1)

- **Found:** Yes
- **Fixed:** Yes — added `mean_squared_error` to the import line
- **Explanation provided:** Yes — "The notebook attempted to compute RMSE using `np.sqrt(mean_squared_error(...))`, but `mean_squared_error` was never imported from `sklearn.metrics`."
- **Impact explained:** Yes — "The kernel would crash at the evaluation cell, preventing any model evaluation metrics from actually being reported."

### Bug 3: Target Leakage — `price` in X (cell-9)

- **Found:** Yes
- **Fixed:** Yes — changed `X = df.copy()` to `X = df.drop(columns=['price'])`
- **Explanation provided:** Yes — "The code copied the entire dataframe: `X = df.copy()`, meaning the target variable price was included in the features."
- **Impact explained:** Yes — "It will yield synthetically perfect predictions (R² ≈ 1.0), making the model entirely misleading and useless for real-world predictions."

### Bug 4: Wrong Column Name — `property_type_raw` (cell-7)

- **Found:** Yes
- **Fixed:** Yes — changed `property_type_raw` to `property_type`
- **Explanation provided:** Yes — "The categorical columns list referenced 'property_type_raw'. However, the actual column is simply 'property_type'."
- **Impact explained:** Yes — "LabelEncoder throws a KeyError loop, completely halting the encoding process and pipeline execution."

---

## Part E: Debugging Scoring (18/20)

| Criterion | Score | Justification |
|---|---|---|
| **Bug detection** | 5/5 | All 4 bugs found in a single pass, including both silent leakage bugs. |
| **Fix quality** | 5/5 | All fixes are correct and follow best practices. Split-then-scale pattern properly implemented. No stale comments or artefacts left behind. |
| **Explanation quality** | 4/5 | Detailed explanations with impact for all 4 bugs — significantly better than inline comments only. However, explanations were delivered in the chat response, not embedded as markdown cells in the notebook itself. The prompt asked for explanations within the notebook. |
| **Completeness** | 4/5 | Runs cleanly end-to-end. Produces valid metrics (R² = 0.277, confirming target leakage removed). Includes predicted vs actual scatter plot and feature importance chart. Minor: no verification cell explicitly proving the fix (e.g., asserting price is not in X.columns). |

### Fixed Notebook Metrics

| Metric | Value |
|---|---|
| RMSE | £825,781 |
| MAE | £90,561 |
| R² | 0.2770 |

These are realistic values confirming that target leakage was properly removed (with leakage, R² would be near 1.0).

---

## Comparison with Cursor

| Aspect | Antigravity | Cursor |
|---|---|---|
| Bugs found | 4/4 | 4/4 |
| Fix quality | All correct, no artefacts | All correct, but left stale `# BUG 2` comment |
| Explanation depth | Detailed with impact for all 4 bugs | Inline comments only for 2 of 4 bugs, no impact |
| Explanation location | Chat response (not in notebook) | Inline comments (not in notebook) |
| Score | **18/20** | **15/20** |

**Key differentiator:** Antigravity's explanations were substantially more thorough than Cursor's — it explained *why* each bug matters and what the real-world impact would be (e.g., "R² ≈ 1.0", "overly optimistic test metrics"). Cursor only provided brief inline comments for 2 bugs and no impact analysis.

---

## Notes

- The original broken notebook contained inline comments hinting at bugs 1, 3, and 4 (e.g., `# BUG 4: References 'property_type_raw'...`), and a note for bug 2 (`# Note: mean_squared_error is used below but not imported here`). This made detection easier than a truly unlabeled broken pipeline.
- Despite the hints, Antigravity's response demonstrated genuine understanding of each bug's statistical and practical impact, rather than mechanical fix-and-move-on behaviour.
- Both tools produced nearly identical fixed metrics (Antigravity R²=0.2770 vs Cursor R²=0.2756), confirming consistent fixes.
