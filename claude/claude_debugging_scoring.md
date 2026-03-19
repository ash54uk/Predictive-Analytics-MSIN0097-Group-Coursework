# Claude Code — Debugging Scoring

**Tool:** Claude Code (VS Code extension)  
**Task:** Find and fix all bugs in `broken_pipeline.ipynb`  
**First-attempt success:** Yes  


---

## Bugs Found (4/4)

### Bug 1 — Scaler fitted before train/test split (data leakage)

- **Location:** Cell 11
- **What:** `StandardScaler.fit_transform()` was called on the entire dataset before `train_test_split`. The scaler computed mean/std using test-set rows too.
- **Impact:** The test set "bled" its statistics into the training data, making evaluation metrics over-optimistic. The model would appear to generalise better than it actually does in production.
- **Fix:** Split first, then `fit_transform` on `X_train` only and `transform` (not `fit_transform`) on `X_test`.

### Bug 2 — `mean_squared_error` not imported (NameError crash)

- **Location:** Cell 1 (imports) and Cell 15 (evaluation)
- **What:** `mean_squared_error` was used in the evaluation cell but never imported — the comment in cell 1 explicitly flagged this omission.
- **Impact:** The notebook crashes with `NameError: name 'mean_squared_error' is not defined`, so no evaluation results are produced at all.
- **Fix:** Added `mean_squared_error` to the `from sklearn.metrics import ...` line.

### Bug 3 — Target column `price` included in features (target leakage)

- **Location:** Cell 9
- **What:** `X = df.copy()` copied the entire dataframe including the `price` column, then `y = df['price']` set the target. The model was trained with the answer as one of its input features.
- **Impact:** This is catastrophic leakage — the model trivially learns to use `price` to predict `price`, producing a near-perfect R² that is completely meaningless. The model would be useless on real new data.
- **Fix:** `X = df.drop(columns=['price'])` to exclude the target from features.

### Bug 4 — Wrong column name `property_type_raw` (KeyError crash)

- **Location:** Cell 7
- **What:** The categorical encoding loop referenced `'property_type_raw'`, but the actual column in the dataset is `'property_type'`.
- **Impact:** The notebook crashes with `KeyError: 'property_type_raw'` at the encoding step, so nothing beyond that cell runs.
- **Fix:** Changed `'property_type_raw'` → `'property_type'` in `categorical_cols`.

---

## Part E: Debugging (20/20)

| Criterion | Score | Justification |
|-----------|-------|---------------|
| Bug detection | 5 | All 4 bugs found: missing import, wrong column name, target leakage (price in X), data leakage (scaler fit before split). Identified both crash bugs and both silent bugs. |
| Fix quality | 5 | All fixes correct and follow best practices. Split-then-fit for scaler, `df.drop(columns=['price'])` for target leakage, correct column name, added import. |
| Explanation quality | 5 | Each bug explained with what it is AND why it matters. Explicitly described impact: e.g., "catastrophic leakage — the model trivially learns to use price to predict price, producing a near-perfect R² that is completely meaningless." |
| Completeness | 5 | Fixed notebook runs cleanly end-to-end without errors. Produces valid model evaluation results. |

---

## Comparison Note

The original `broken_pipeline.ipynb` contained `# BUG` comments labelling the intentional bugs, which reduced the difficulty of detection. Despite this, the quality of explanations and fixes varied significantly across tools — Claude Code provided the most detailed impact analysis of all tools tested.
