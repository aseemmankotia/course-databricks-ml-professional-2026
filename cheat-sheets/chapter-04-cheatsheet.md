# Feature Engineering in Unity Catalog — Cheat Sheet

## Client
```python
from databricks.feature_engineering import FeatureEngineeringClient, FeatureLookup, FeatureFunction
fe = FeatureEngineeringClient()
```

## Create / Write Table
| Call | Required params | Notes |
|---|---|---|
| `fe.create_table` | `name`, `primary_keys`, `df` or `schema` | `timestamp_keys` optional → makes it a **time-series** table |
| `fe.write_table` | `name`, `df`, `mode` | `mode='merge'` (upsert by PK) or `mode='overwrite'` (replace all) |

## Feature Table Types
| Type | Source | Refresh | Use case |
|---|---|---|---|
| Batch | Static/scheduled job | Periodic (e.g. nightly) | Aggregates, slowly changing features |
| Streaming | Structured Streaming `readStream` | Continuous | Near real-time features |
| On-demand | UC Python function (`FeatureFunction`) | At request time | Needs request-only inputs; no leakage b/w train & serve |

## Point-in-Time Join
```python
lookup = FeatureLookup(
    table_name="catalog.schema.feat_tbl",
    feature_names=["f1","f2"],
    lookup_key="id",
    timestamp_lookup_key="event_ts"  # enables as-of join
)
training_set = fe.create_training_set(
    df=labels_df,
    feature_lookups=[lookup],
    label="target",
    exclude_columns=["event_ts"]
)
training_df = training_set.load_df()
```
Rule: as-of join returns the **latest feature row with timestamp <= label's timestamp** → prevents future leakage.

## On-Demand Features
```python
fn = FeatureFunction(
    udf_name="catalog.schema.calc_distance",
    input_bindings={"lat1":"lat","lon1":"lon","lat2":"req_lat","lon2":"req_lon"},
    output_name="distance"
)
```
Computed identically at **train time** (over historical df) and **serve time** (over request payload) → zero train/serve skew.

## Avoiding Leakage — Checklist
1. Use `timestamp_keys` on feature table + `timestamp_lookup_key` on FeatureLookup.
2. Pass `exclude_columns` to drop timestamp/id helper cols before fitting.
3. Never manually join feature tables with a plain equality join on ID alone if timestamps matter.

## Governance
- All feature tables, training sets, and models registered in UC → automatic **lineage graph** in Catalog Explorer (table → notebook/job → model).
- Access control via standard UC GRANT/REVOKE on catalog.schema.table.

## Common Exam Traps
- Confusing `write_table` (writes data) with `create_table` (defines schema/keys) — keys can't be added after creation without recreating.
- Choosing streaming/batch table for a feature that only exists at inference request time (should be on-demand).
- Forgetting `exclude_columns`, causing timestamp/label columns to leak into `training_df`.
