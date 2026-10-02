## Chapter 9: Lakehouse Monitoring: Detecting Data and Model Drift — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | Which statistical test does Lakehouse Monitoring use for continuous/numeric feature drift? | Kolmogorov-Smirnov (KS) test — compares empirical CDFs of two samples, producing a statistic + p-value. Exam tip: continuous = KS. |
| 2 | Which statistical test does Lakehouse Monitoring use for categorical feature drift? | Chi-squared test — compares frequency distributions of categories. Exam tip: categorical/string = chi-squared. |
| 3 | Three Lakehouse Monitor profile types? | TimeSeries (needs timestamp_col), Snapshot (no time dimension, full table each refresh), InferenceLog (model serving inference tables, needs prediction_col/model_id_col/timestamp_col). |
| 4 | Required parameters for an InferenceLog monitor (excluding optional ones)? | problem_type ('classification'\|'regression'), prediction_col, timestamp_col, model_id_col. label_col and baseline_table_name are optional. |
| 5 | What does setting label_col on an InferenceLog monitor enable? | Prediction-vs-ground-truth drift comparison and accuracy/performance metrics. Labels must be joined into the inference table manually — the monitor does not fetch them automatically. |
| 6 | Does a streaming Model Serving endpoint get continuous, real-time drift metrics? | No. Refreshes are always scheduled or manually triggered batch computations (via MonitorCronSchedule or lm.run_refresh), regardless of batch vs streaming endpoint. |
| 7 | Command to create a monitor in Python? | `import databricks.lakehouse_monitoring as lm; lm.create_monitor(table_name=..., profile_type=lm.InferenceLog(...), output_schema_name=...)` |
| 8 | Command to manually trigger a monitor refresh? | `lm.run_refresh(table_name="catalog.schema.table")` |
| 9 | What does the profile_metrics table contain? | Per-column summary statistics (count, nulls, mean, distinct count, etc.) computed for each time window/granularity — descriptive, not comparative. |
| 10 | What does the drift_metrics table contain? | Per-column comparison statistics between two windows: 'CONSECUTIVE' (vs prior window) or 'BASELINE' (vs baseline_table_name) — includes KS/chi-squared statistic + p-value. |
| 11 | What optional parameter enables comparing current data to a fixed reference dataset instead of the previous window? | baseline_table_name — when set, drift_metrics includes a 'BASELINE' comparison row type in addition to 'CONSECUTIVE'. |
| 12 | Which monitor type is best for a static dimension table with no timestamp column? | Snapshot — evaluates the full table each refresh with no time-window bucketing required. |
