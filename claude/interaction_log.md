# AI Interaction Log — Manchester Property Price Prediction
**Date:** 2026-03-18 | **Tool:** Claude Code

---

| # | User Prompt | Action Taken | Outcome |
|---|---|---|---|
| 1 | Build a full end-to-end Jupyter notebook: ingestion, EDA, modelling, improvement on `manchester_features.parquet` | Created `manchester_property_analysis.ipynb` — 76 cells covering all four pipeline stages | Y |
| 2 | Getting an error when I run the load and inspect code | Diagnosed as path issue; added `_HERE` directory resolution | N — real cause was pyarrow bug |
| 3 | Still getting error after adding HERE | Identified pyarrow histogram bug; added `engine='fastparquet'` | N — kernel not restarted |
| 4 | Still throws error after installing fastparquet | Re-applied fix; advised kernel restart | N — stale kernel |
| 5 | Just use `pd.read_parquet` without the engine parameter | Removed engine parameter as instructed | N — defaults back to broken pyarrow |
| 6 | Still getting an error | Offered two options: upgrade pyarrow or use fastparquet | N — kernel not restarted after upgrade |
| 7 | There exists a manchester data cleaning notebook, what is that? | Checked directory — only `broken_pipeline.ipynb` (supplied) and the built notebook exist | Y |
| 8 | Check the notebook, still throwing errors | Discovered `manchester_features.parquet` is a **nested Spark-partitioned directory**, not a single file. Corrected path to inner directory | N — old pyarrow in kernel |
| 9 | Still getting same error | Tested fastparquet on correct inner path from terminal — confirmed working. Updated load cell | N — unconfirmed |
| 10 | Is the notebook updated to run again? | Read all 76 cells, confirmed structure complete | — |
| 11 | Still getting the same error for load and inspect | Re-applied `engine='fastparquet'` with correct nested path definitively | N — kernel issue |
| 12 | Clarified: file is Spark directory, try `pyarrow.parquet.read_table` or concat parts | Tested both approaches; confirmed fastparquet works; updated cell | Y — notebook ran end-to-end |
| 13 | `NameError: savefig is not defined` | Re-added `savefig` helper to cell 1 (dropped during earlier edits) | Y |

---

## Final Results

| Model | MAE | RMSE | R² | MdAPE |
|---|---|---|---|---|
| Ridge (baseline) | £60,340 | £95,776 | 0.52 | 22.8% |
| Random Forest | £40,237 | £70,684 | 0.75 | 13.7% |
| XGBoost | £41,237 | £71,272 | 0.75 | 14.4% |

**Key findings:** Location (lat/lon) is the dominant price driver. New-build premium = 27.6%. Floor area correlation = 0.44. Transaction volume dipped in 2020 (COVID) and peaked in 2021 (stamp-duty holiday).

**Total exchanges:** 13 | **Notebook cells:** 76 | **Dataset:** 474,144 transactions
