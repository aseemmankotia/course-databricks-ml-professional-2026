## Chapter 10 Cheat Sheet — Alerting & Production Governance for ML Systems

### Pipeline Roles
| Component | Role |
|---|---|
| Lakehouse Monitoring | Computes profile + drift metrics into Delta tables (js_distance, chi-squared p-value, KS-test, percent_null) |
| Databricks SQL Alert | Queries a metrics table on a schedule; fires notification/webhook when condition is true |
| Webhook -> Job | Bridges alert to action: triggers a retraining Job via the Jobs API |
| Job (retrain -> validate -> promote) | Executes retraining, evaluates vs current Champion, conditionally updates alias |
| Unity Catalog alias | Mutable pointer (e.g., Champion/Challenger) to a specific model version — replaces old Staging/Production stages |
| UC Audit Logs | Governance record: who moved which alias, when, and permission changes |
| Asset Bundle (`databricks.yml`) | Defines Job/monitor/model resources as code; deployed via `databricks bundle deploy` |

### Escalation Strategy (memorize order)
**Alert -> Investigate -> Retrain -> Validate -> Promote**
Any answer that skips Validate before Promote is wrong.

### Scenario Language Decoder
| Scenario phrase | Maps to |
|---|---|
| "react automatically to a live metric threshold" | Monitoring + Automation (alerting) |
| "gate before deployment / must pass checks first" | Testing |
| "move version between stages / update alias" | Lifecycle |
| "who approved / audit record" | Governance (UC audit logs) |
| "reproducible pipeline across environments" | Automation (Asset Bundles / Jobs) |

### Key Commands
```python
# Point an alias at a validated version (UC-registered model)
from mlflow import MlflowClient
client = MlflowClient()
client.set_registered_model_alias(name="catalog.schema.model", alias="Champion", version=5)
```
```yaml
# databricks.yml (Asset Bundle) — retrain job skeleton
resources:
  jobs:
    retrain_validate_promote:
      tasks:
        - task_key: retrain
        - task_key: validate
          depends_on: [{task_key: retrain}]
        - task_key: promote
          depends_on: [{task_key: validate}]
```

### Common Traps
- SQL Alerts evaluate on **schedule**, not on every table write.
- SQL Alerts **cannot** directly call MLflowClient — they only notify; a Job/script must act.
- UC-registered models use **aliases**, not `transition_model_version_stage` (that API targets the legacy workspace registry).
- Promotion without validation = automatic wrong answer.
- "Monitoring detects, alerting reacts, testing gates, lifecycle governs" — keep these four buckets separate when mapping scenario language.
