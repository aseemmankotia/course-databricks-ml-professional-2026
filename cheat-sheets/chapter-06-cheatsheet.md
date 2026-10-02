## Chapter 6 Cheat Sheet — Models in Unity Catalog

### Registry Comparison
| Feature | Legacy Workspace Registry | Unity Catalog Registry |
|---|---|---|
| Namespace | single model name | catalog.schema.model |
| Scope | workspace-only | metastore (cross-workspace) |
| Version pointer | Stages (Staging/Production/Archived) | Aliases (Champion, Challenger, custom) |
| ACLs | workspace permissions | fine-grained UC grants (USE CATALOG/SCHEMA, EXECUTE, etc.) |
| Lineage | limited | automatic, native UC lineage to data/notebooks/runs |
| Status | deprecated for new use | current recommended approach |

### Required Setup
```python
import mlflow
mlflow.set_registry_uri("databricks-uc")
mlflow.register_model(model_uri, "catalog.schema.model_name")
```
Without `set_registry_uri("databricks-uc")`, models register to the legacy workspace registry.

### Aliases — Core Commands
```python
from mlflow import MlflowClient
client = MlflowClient()

# Assign/reassign alias
client.set_registered_model_alias(name="catalog.schema.model", alias="Champion", version=3)

# Remove alias
client.delete_registered_model_alias(name="catalog.schema.model", alias="Challenger")

# Get version by alias
mv = client.get_model_version_by_alias("catalog.schema.model", "Champion")
```

### Loading Models
| Method | URI |
|---|---|
| By alias (UC) | `models:/catalog.schema.model@Champion` |
| By version | `models:/catalog.schema.model/3` |
| By stage (legacy only, NOT UC) | `models:/name/Production` |

### Aliases vs Stages
- Stages: fixed set (Staging/Production/Archived), single active assignment per stage per model, workspace-scoped.
- Aliases: arbitrary names, multiple aliases can exist, each points to exactly one version, catalog-scoped, mutable/reassignable instantly, no separate history log — audit comes from UC lineage + model version history.

### Governance Workflow (Champion/Challenger)
1. Train → log run → `mlflow.register_model()` creates new version.
2. Assign `Challenger` alias to new version for evaluation.
3. Validate against production traffic/metrics.
4. On success, reassign `Champion` alias to that version.
5. Old Champion version remains registered (no alias) — fully retrievable via version number, preserving lineage.

### Permissions (key privileges)
| Privilege | Grants |
|---|---|
| USE CATALOG / USE SCHEMA | required to reference parent containers |
| CREATE MODEL | create new registered models in a schema |
| EXECUTE | load/use a model version for inference |
| Ownership / broader grants | modify metadata, set/change aliases and tags |

### Cross-Workspace Sharing
Any workspace attached to the same Unity Catalog metastore can access a registered model automatically once granted privileges — no export/copy/re-registration required.

### Comparing Versions
```python
client.search_model_versions("name='catalog.schema.model'")
```
Use MLflow UI **Compare Runs** to diff metrics/params of the training runs linked to each version before promoting.
