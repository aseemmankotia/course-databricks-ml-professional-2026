# Chapter 7 Cheat Sheet: CI/CD for ML with Databricks Asset Bundles

## databricks.yml Structure
| Key | Purpose |
|---|---|
| bundle | Identity/name of the bundle |
| include | Split config across multiple yaml files |
| variables | Bundle-level parameters with defaults |
| resources | Jobs, pipelines, experiments, models, serving endpoints to deploy |
| targets | Per-environment overrides (workspace, mode, run_as, variables) |

## Target Modes
| Mode | Behavior |
|---|---|
| development | Prefixes resource names with user identity; pauses schedules/triggers by default; isolates dev runs |
| production | Deploys exactly as configured; recommend run_as: service principal; schedules/triggers active |

## CLI Commands
| Command | Purpose |
|---|---|
| databricks bundle init | Scaffold new bundle from template |
| databricks bundle validate -t <target> | Check syntax/config before deploy |
| databricks bundle deploy -t <target> | Push resources to workspace target |
| databricks bundle run -t <target> <job-key> | Trigger a deployed job |
| databricks bundle destroy -t <target> | Tear down deployed resources |

## Parameterizing Across Environments
```yaml
variables:
  cluster_size:
    default: "Standard_DS3_v2"
targets:
  dev:
    variables:
      cluster_size: "Standard_DS3_v2"
  prod:
    variables:
      cluster_size: "Standard_DS5_v2"
resources:
  jobs:
    train_job:
      tasks:
        - task_key: train
          new_cluster:
            node_type_id: ${var.cluster_size}
```

## Retraining Triggers
| Trigger Type | Use Case |
|---|---|
| Scheduled (cron) | Fixed cadence retraining (e.g. nightly) |
| File arrival | Retrain when new data lands in a storage path |
| Continuous | Rerun immediately after prior run completes (rare for training) |
| Drift-triggered | Monitoring job detects drift breach → conditionally launches retrain job via API/task |

## Multi-Task Job Orchestration Pattern
train_task → evaluate_task → register_task → deploy_task
- Use depends_on with default run_if: ALL_SUCCESS
- evaluate_task raises/fails if metric below threshold → downstream tasks auto-skipped

## Champion/Challenger Auto-Promotion
1. Retraining job registers new version, sets alias @challenger
2. Evaluation task pulls @champion and @challenger metrics via MLflow API
3. Compare against threshold (e.g., >=2% improvement)
4. If pass: reassign @champion alias to challenger version via Registry API
5. If fail: leave @champion unchanged, log result

## Exam Traps
- Continuous trigger ≠ event-driven data-arrival trigger
- Omitting -t target flag in CI scripts risks deploying to the wrong workspace
- "Always promote the latest version" is NOT automated promotion logic — it skips validation
- Removing task dependencies to "simplify" breaks the eval → deploy gating

## CI/CD Git Flow
- PR → main: validate + deploy staging target + run integration test job
- Git tag/release: deploy prod target (run_as service principal recommended)
