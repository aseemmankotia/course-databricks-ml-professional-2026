## Deployment Strategies

| Strategy | Traffic behavior | User impact | Rollback speed |
|---|---|---|---|
| Blue-Green | 100% instant cutover after validation | Full exposure at cutover | Instant (switch back) |
| Canary | Gradual % increase to new model | Partial, growing exposure | Fast (reduce %) |
| Shadow | Mirrored copy, no response returned to user | None | N/A (never user-facing) |

**Exam trap:** 'Compare predictions without affecting users' → Shadow, NOT canary. 'Instant full switch after validation' → Blue-green, NOT canary.

## Serving Pattern Selection

| Requirement | Pattern | Implementation |
|---|---|---|
| Large offline scoring, no latency need | Batch | `mlflow.pyfunc.spark_udf` in scheduled Spark job |
| Unbounded stream, seconds-level latency | Streaming | Structured Streaming + `spark_udf`, often `foreachBatch` → Delta |
| Synchronous, low-latency per-request | Real-time | Model Serving endpoint, REST `/serving-endpoints/{name}/invocations` |

## Endpoint Traffic Config (Canary Example)
```json
{
  "served_entities": [
    {"name": "model_v3", "entity_name": "catalog.schema.model", "entity_version": "3"},
    {"name": "model_v4", "entity_name": "catalog.schema.model", "entity_version": "4"}
  ],
  "traffic_config": {
    "routes": [
      {"served_model_name": "model_v3", "traffic_percentage": 90},
      {"served_model_name": "model_v4", "traffic_percentage": 10}
    ]
  }
}
```
Blue-green = same config but set new version to 100% in one update (no intermediate %).

## Unity Catalog PyFunc Serving
- Register: `mlflow.register_model(model_uri, "catalog.schema.model_name")`
- Reference UC model + version directly as `served_entities[].entity_name` / `entity_version` — no flavor conversion needed; PyFunc is natively supported.

## Monitoring Checklist

| Metric | What it signals | Exam angle |
|---|---|---|
| Latency (p50/p90/p99) | Response time distribution | Sudden p99 spike + stable QPS → check scaling/cold start |
| QPS | Request throughput | Compare against provisioned capacity |
| Error rate (4xx/5xx) | Client vs server failures | 5xx spike + latency spike → resource/scaling issue |
| Resource utilization / scale-to-zero | Compute provisioning | Cold starts from scale-to-zero cause correlated latency+error spikes |

**Common mistake:** Assuming a canary traffic-config change explains an error spike when QPS is flat — check compute scaling/cold starts first.
