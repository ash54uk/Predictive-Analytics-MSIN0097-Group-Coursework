# Benchmark Prompt — Single Notebook, Open-Ended

All 3 tools (Antigravity, Claude Code, Cursor) receive this **exact same prompt**.
Each tool produces **one notebook** covering the entire pipeline.

---

## The Prompt

```
You are given a parquet file called `manchester_features.parquet` containing
property transaction data for Greater Manchester (2015–2024). Your task is to
produce a single, end-to-end Jupyter notebook that covers the full data science
pipeline:

1. **Data Ingestion & Cleaning**
   - Load the dataset and explore its structure
   - Assess data quality: missing values, outliers, uninformative columns
   - Clean the data, documenting and justifying every decision you make

2. **Exploratory Data Analysis**
   - Produce visualisations that reveal the key patterns and relationships
     in the data
   - Write markdown commentary explaining what each visualisation shows
     and why it matters

3. **Predictive Modelling**
   - Build a model to predict property sale price
   - Choose your own approach: model type, feature engineering, encoding
     strategy, and evaluation method
   - Justify your choices
   - Report evaluation metrics and interpret the results
   - Visualise model performance

4. **Model Improvement**
   - Attempt to improve on your initial model using any strategy you
     see fit (feature engineering, different algorithm, tuning, etc.)
   - Compare the improved model against the baseline quantitatively
   - Explain what drove the improvement

The notebook should:
- Run end-to-end without errors
- Set random_state=42 wherever randomness is involved
- Include a requirements section in the first cell
- Contain markdown cells explaining your reasoning throughout
- Be reproducible — another person should be able to re-run it and
  get the same results
```

---

## After the Main Notebook: Debugging Task

Once the main notebook is complete, give each tool the `broken_pipeline.ipynb`
file with this prompt:

```
The attached notebook `broken_pipeline.ipynb` contains a ML pipeline for
predicting property prices using the Manchester property dataset. The notebook
has several bugs that prevent it from running correctly or produce invalid
results.

Find and fix all the bugs. For each bug:
- Explain what the bug is
- Explain why it matters (what impact it has on results)
- Show the fix

Ensure the fixed notebook runs end-to-end without errors and produces
valid model evaluation results.
```
