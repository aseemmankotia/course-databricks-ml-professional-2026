## Chapter 9: Lakehouse Monitoring: Detecting Data and Model Drift — Practice Questions

**Q1.** A model serving endpoint logs 'account_age_days' (a numeric feature) to its inference table. Which statistical test does Databricks Lakehouse Monitoring use to detect drift on this column?

A) Chi-squared test
B) Z-test for proportions
C) Kolmogorov-Smirnov (KS) test
D) ANOVA

<details>
<summary>Answer</summary>

**Correct: C**

Lakehouse Monitoring runs the two-sample KS test on numeric/continuous columns, comparing the empirical CDF of the current window against the baseline or previous window.

**Why the others are wrong:**
- Chi-squared is applied only to categorical/string columns, not continuous numeric data.
- Z-test for proportions is not part of Lakehouse Monitoring's drift metric computation.
- ANOVA is not used by Lakehouse Monitoring for drift detection.

</details>

---

**Q2.** An inference table includes a 'region' column (string values like 'US', 'EU', 'APAC'). Which test appears in the drift_metrics table for this column?

A) Chi-squared test
B) Kolmogorov-Smirnov test
C) Kendall's tau
D) T-test

<details>
<summary>Answer</summary>

**Correct: A**

Chi-squared compares frequency distributions between two categorical samples, which is exactly what Lakehouse Monitoring uses for string/categorical columns.

**Why the others are wrong:**
- KS test is reserved for continuous numeric columns.
- Kendall's tau is a correlation measure, not used by Lakehouse Monitoring.
- T-test compares means of continuous data, not categorical frequencies.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A team wants to monitor a Delta table auto-populated by a Model Serving endpoint's request/response logging, tracking model accuracy over time as ground truth becomes available. Which monitor profile type should they use?

A) Snapshot
B) InferenceLog
C) TimeSeries
D) Custom SQL profile

<details>
<summary>Answer</summary>

**Correct: B**

InferenceLog is the profile type designed for model serving inference tables — it requires prediction_col, timestamp_col, model_id_col, and optionally label_col to compute prediction drift and performance metrics.

**Why the others are wrong:**
- Snapshot is for tables without a time dimension, refreshed in full each time — not suited to tracking model performance over time.
- TimeSeries is for general time-indexed tables without model-specific metrics like accuracy or label drift.
- There is no 'Custom SQL profile' monitor type in Lakehouse Monitoring.

</details>

---

**Q4.** A Model Serving endpoint is configured for real-time streaming inference, and its requests/responses are captured into a Delta inference table. How does Lakehouse Monitoring compute drift metrics for this table?

A) It runs a continuous streaming job that updates drift metrics on every new row
B) It only supports drift metrics for batch endpoints, not streaming endpoints
C) It runs a scheduled or manually triggered batch refresh over the table, regardless of how the table is populated
D) It automatically refreshes drift metrics every time the endpoint receives a request

<details>
<summary>Answer</summary>

**Correct: C**

Monitor refresh in Lakehouse Monitoring is always a scheduled or manually triggered batch computation (via MonitorCronSchedule or lm.run_refresh), independent of whether the source data arrives via batch jobs or a streaming endpoint.

**Why the others are wrong:**
- Lakehouse Monitoring does not offer a continuous/streaming refresh mode — all refreshes are batch jobs.
- InferenceLog monitors work identically for batch and streaming endpoints; the underlying mechanism is the same.
- Refreshes are decoupled from individual endpoint requests — they run on a schedule/trigger, not per-request.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** When creating an InferenceLog monitor with lm.create_monitor(), which parameters are required at minimum (ignoring baseline and label options)?

A) problem_type, label_col, baseline_table_name
B) timestamp_col, granularities, output_schema_name only
C) problem_type, prediction_col, timestamp_col, model_id_col
D) prediction_col and label_col only

<details>
<summary>Answer</summary>

**Correct: C**

The InferenceLog profile requires problem_type ('classification' or 'regression'), prediction_col, timestamp_col, and model_id_col to identify predictions across model versions and time windows.

**Why the others are wrong:**
- label_col and baseline_table_name are optional — label_col enables label/prediction drift, baseline_table_name enables baseline comparison, but neither is mandatory.
- granularities and output_schema_name are needed but insufficient alone — core InferenceLog identity columns are still required.
- prediction_col alone is required, but label_col is optional, and this option omits required fields like timestamp_col and model_id_col.

</details>

---

**Q6.** A fraud detection model's inference table has prediction_col set but no label_col configured on the monitor. The team wants to see prediction-vs-actual drift and accuracy metrics next month once true fraud outcomes are confirmed. What must they do?

A) Nothing — Lakehouse Monitoring automatically fetches ground truth labels from any linked feature table
B) Delete and recreate the inference table with the labels baked in from day one — retroactive labeling is not supported
C) Enable 'auto-label' in the endpoint configuration
D) Join the confirmed ground-truth labels into the inference table (or a table it references), then update the monitor to set label_col before refreshing

<details>
<summary>Answer</summary>

**Correct: D**

Lakehouse Monitoring does not source ground-truth labels itself — the team must join actual outcomes into the inference table (or a table the monitor reads), then configure label_col so drift/accuracy metrics compare predictions to labels on the next refresh.

**Why the others are wrong:**
- Lakehouse Monitoring has no automatic mechanism to fetch ground truth from feature tables — it only reads columns present in the monitored table.
- Labels can be added retroactively via a join; recreating the table is unnecessary.
- There is no 'auto-label' setting on Model Serving endpoints.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** What is the key difference between the profile_metrics table and the drift_metrics table generated by a Lakehouse Monitor?

A) profile_metrics stores raw request payloads; drift_metrics stores model predictions
B) profile_metrics is generated only for InferenceLog monitors; drift_metrics is generated only for Snapshot monitors
C) They contain identical data but profile_metrics is for numeric columns and drift_metrics is for categorical columns
D) profile_metrics summarizes per-column statistics for each time window; drift_metrics compares those statistics between two windows (consecutive or baseline) to detect change

<details>
<summary>Answer</summary>

**Correct: D**

profile_metrics captures descriptive statistics (count, nulls, mean, distinct values, etc.) per column per window/granularity; drift_metrics computes comparison statistics (KS test, chi-squared test) between consecutive windows or against a baseline.

**Why the others are wrong:**
- Neither table stores raw payloads — both store computed statistics, not raw data.
- Both tables are generated for all monitor profile types, not restricted to specific types.
- The column-type split (numeric vs categorical) applies to which test runs within drift_metrics, not to which table is generated.

</details>

---

**Q8.** A data engineering team maintains a static dimension table with no timestamp column and wants periodic drift checks on its full contents after each nightly reload. Which monitor type is most appropriate?

A) Snapshot
B) InferenceLog
C) TimeSeries
D) Streaming

<details>
<summary>Answer</summary>

**Correct: A**

Snapshot monitors evaluate the entire table content on each refresh without requiring a timestamp column, making them ideal for dimension/reference tables reloaded periodically.

**Why the others are wrong:**
- InferenceLog requires model serving-specific columns (prediction_col, model_id_col) that don't apply to a generic dimension table.
- TimeSeries requires a timestamp_col to bucket data into time windows, which this table lacks.
- 'Streaming' is not a Lakehouse Monitoring profile type.

</details>

---

