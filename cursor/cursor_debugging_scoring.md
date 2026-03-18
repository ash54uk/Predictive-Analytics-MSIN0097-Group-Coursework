# Cursor — Debugging Task Scoring

**Notebook:** `broken_pipeline.ipynb` (fixed version)
**Original:** `broken_pipeline_before_fix.ipynb`
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

## What Cursor Found and Fixed

### Bug 1: Data Leakage — Scaler fit before split (cell-11)

- **Found:** Yes
- **Fixed:** Yes — reordered to split first, then `fit_transform` on train and `transform` on test
- **Explanation provided:** Inline comment only: `# Split first, then fit scaler only on training data to avoid data leakage`
- **Impact explained:** No

### Bug 2: Missing Import — `mean_squared_error` (cell-1)

- **Found:** Yes
- **Fixed:** Yes — added `mean_squared_error` to the import line
- **Explanation provided:** No (stale comment `# BUG 2: mean_squared_error was never imported — this cell will crash` left in cell-15 from the original)
- **Impact explained:** No

### Bug 3: Target Leakage — `price` in X (cell-9)

- **Found:** Yes
- **Fixed:** Yes — changed `X = df.copy()` to `X = df.drop(columns=['price']).copy()`
- **Explanation provided:** Inline comment: `# Define features and target — exclude price from X to avoid target leakage`
- **Impact explained:** No (does not mention that this would inflate R² to near 1.0)

### Bug 4: Wrong Column Name — `property_type_raw` (cell-7)

- **Found:** Yes
- **Fixed:** Yes — changed `property_type_raw` to `property_type`
- **Explanation provided:** No
- **Impact explained:** No

---

## Part E: Debugging Scoring (15/20)

| Criterion | Score | Justification |
|---|---|---|
| **Bug detection** | 5/5 | All 4 bugs found, including both silent leakage bugs. |
| **Fix quality** | 4/5 | All fixes are correct and follow best practices. Minor issue: left stale `# BUG 2` comment in cell-15 of the fixed version. |
| **Explanation quality** | 2/5 | No dedicated markdown cells explaining each bug as the prompt requested. Only brief inline comments for 2 of 4 bugs. Does not explain impact (e.g. "target leakage inflates R² to near 1.0"). |
| **Completeness** | 4/5 | Runs cleanly end-to-end. Produces valid metrics. Leftover stale comment is cosmetic but sloppy. |

### Fixed Notebook Metrics

| Metric | Value |
|---|---|
| RMSE | £826,527 |
| MAE | £90,708 |
| R² | 0.2756 |

These are realistic values confirming that target leakage was properly removed (with leakage, R² would be near 1.0).

---

## Notes

- The original broken notebook contained inline comments labeling bugs 1, 3, and 4 (e.g. `# BUG 4: References 'property_type_raw'...`), and a hint for bug 2 (`# Note: mean_squared_error is used below but not imported here`). This made detection easier than a truly unlabeled broken pipeline.
- Despite this, Cursor did not produce the explanation markdown cells that the prompt explicitly asked for ("For each bug: explain what the bug is, explain why it matters, show the fix").
- The stale `# BUG 2` comment left in the fixed notebook suggests the fixes were made mechanically without a full review pass.
