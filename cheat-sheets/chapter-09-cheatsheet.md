## Lakehouse Monitoring — Chapter 9 Cheat Sheet

### Monitor Types
| Type | Use Case | Requires |
|---|---|---|
| TimeSeries | Time-indexed tables, general drift over time | timestamp_col |
| Snapshot | Static/dimension tables, full-table refresh | nothing (no time column) |
| InferenceLog | Model Serving inference tables (batch OR streaming) | prediction_col, timestamp_col, model_id_col, problem_type |

### Drift Tests by Data Type
| Data Type | Test | What it compares |
|---|---|---|
| Continuous/numeric | Kolmogorov-Smirnov (KS) | Empirical CDFs → statistic + p-value |
| Categorical/string | Chi-squared | Category frequency distributions |

### InferenceLog Params
- **Required:** problem_type ("classification"/"regression"), prediction_col, timestamp_col, model_id_col
- **Optional:** label_col (enables label/prediction drift + accuracy metrics), baseline_table_name (enables BASELINE comparison rows), granularities (e.g. ["1 day"])
- Ground-truth labels are NOT auto-fetched — join them into the inference table yourself, then set label_col.

### Generated Assets (all monitor types)
- `<table>_profile_metrics` — per-column descriptive stats per window/granularity (count, nulls, mean, distinct, etc.)
- `<table>_drift_metrics` — per-column comparison stats: drift_type = CONSECUTIVE (vs previous window) or BASELINE (vs baseline_table_name); contains ks_test.statistic/pvalue for numeric, chi_squared_test.statistic/pvalue for categorical
- Auto-generated Databricks SQL dashboard attached to the monitor

### Refresh Behavior — KEY GOTCHA
- Monitor refresh is ALWAYS a scheduled or manually triggered BATCH job.
- Applies equally to batch and streaming Model Serving endpoints — the inference table's population method (batch write vs streaming append) does not change how the monitor computes metrics.
- Trigger via `MonitorCronSchedule` (scheduled) or `lm.run_refresh(table_name=...)` (manual).

### Commands
```python
import databricks.lakehouse_monitoring as lm

lm.create_monitor(
  table_name="catalog.schema.inference_table",
  profile_type=lm.InferenceLog(
    problem_type="classification",
    prediction_col="prediction",
    label_col="label",
    model_id_col="model_version",
    timestamp_col="timestamp",
    granularities=["1 day"]
  ),
  output_schema_name="catalog.monitoring",
  baseline_table_name="catalog.schema.baseline"
)

lm.run_refresh(table_name="catalog.schema.inference_table")
```

### Exam Gotchas
1. KS = continuous only; chi-squared = categorical only. Don't swap.
2. Streaming endpoint ≠ streaming drift computation — refresh is still batch.
3. label_col requires you to join ground truth in manually; monitor won't fetch it.
4. profile_metrics = descriptive stats per window; drift_metrics = comparison between windows.
5. Snapshot monitors have no time dimension — use baseline_table_name for meaningful drift comparison.
