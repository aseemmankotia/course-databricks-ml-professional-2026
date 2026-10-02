# Chapter 8 Cheat Sheet: Testing ML Pipelines

## Test Type Comparison
| Test Type | Scope | Runs Where | Speed | Verifies |
|---|---|---|---|---|
| Unit | Single function (e.g., feature transform, predict()) | Local Python / local SparkSession, no cluster | Fast (ms–sec) | Logic correctness, output type/shape |
| Integration | Full or scaled-down pipeline (ingest→transform→train→register) | Real Databricks Job, staging target, small sample data | Slower (min) | End-to-end wiring, artifacts produced |

## Unit Testing Patterns
- Extract business logic into **pure Python/Pandas functions**; keep Spark wrapper thin → test logic without a cluster.
- For Spark-required logic: use a **session-scoped local SparkSession** (`master('local[*]')`) fixture with small in-memory DataFrames.
- Use **chispa** (`assert_df_equality`) for readable Spark DataFrame equality checks.
- Never mock away the actual transformation logic just to make a test pass — assert real output values.

## Testing PyFunc predict()
Check ALL of:
1. **Type** — e.g., `isinstance(output, pd.DataFrame)`
2. **Shape** — `output.shape[0] == input_df.shape[0]`
3. **Columns/schema** — expected column names present

```python
output = model.predict(input_df)
assert isinstance(output, pd.DataFrame)
assert output.shape[0] == input_df.shape[0]
assert "prediction" in output.columns
```

## Data Validation Before Training
| Check | Example |
|---|---|
| Schema | `df.schema == expected_schema` |
| Nulls | `df.filter(col("x").isNull()).count() == 0` |
| Ranges | `df.filter((col("age") < 0) \| (col("age") > 120)).count() == 0` |

Fail fast with a clear assertion/exception **before** `.fit()` — do not silently swallow with broad `try/except`.

## Model Quality Gate (Promotion)
1. Load candidate + current champion metrics via MLflow API.
2. Compare programmatically against a defined threshold (e.g., F1 improvement ≥ 0.02).
3. Only if passed → `client.set_registered_model_alias(name, "champion", version)` (Unity Catalog uses **aliases**, not legacy Staging/Production stages).
4. Never auto-approve all promotion requests via webhooks — that bypasses the gate.

## Asset Bundle Testing Workflow
| Step | Command |
|---|---|
| Deploy to staging | `databricks bundle deploy -t staging` |
| Run job in staging | `databricks bundle run -t staging <job-key>` |
| Promote to prod | `databricks bundle deploy -t prod` (only after staging passes) |

- Always validate against the **staging target** — do not repurpose prod's config as a fake staging environment.
- Never validate by running jobs manually from the UI outside the bundle definition — it won't test the actual deployed YAML.

## Integration Test Design
- Parameterize the job with a **scaled-down/sample dataset** (widget or job parameter override) instead of full production data.
- Assert on: job run status (SUCCESS), model version registered, expected metrics logged.
- Avoid: full production-scale runs on every merge (too slow/costly) and skipping integration tests entirely (misses cross-stage wiring bugs).
