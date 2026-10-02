## Chapter 5 Cheat Sheet — Model Development Scenarios

### Decision Tree: Tool Selection by Keyword
| Scenario Keyword | Tool | Why |
|---|---|---|
| Independent groups (one model per store/segment) | **pandas Function API** (applyInPandas/mapInPandas) | Parallelizes many small independent single-node trainings across Spark |
| Single model, many trials, efficient sampling (TPE) | **Optuna** | Databricks-recommended replacement for deprecated Hyperopt |
| Huge search space, needs cluster-wide distribution, early stopping | **Ray Tune** | Distributes trials/training across many nodes with schedulers |
| Single model, algorithm natively parallel | **Spark MLlib** | Distributes the training computation of ONE model |

### Distributed Data Prep vs Distributed Training
| Layer | Tool | Distributes |
|---|---|---|
| Data prep | Spark DataFrame/SQL | Transformations across cluster |
| Model training | Spark MLlib | Training computation of a single model |
| Independent training | pandas Function API | Many independent model fits, one per group |

### Leakage Risk Clues → Feature Engineering Fix
| Clue in Scenario | Correct Approach |
|---|---|
| "Only use data known at prediction time" | Feature Store **point-in-time lookup** (as-of join) |
| "Restore table to prior state/version" | Delta Lake **time travel** (not row-wise, not leakage fix) |
| "Low-latency lookup at inference" | **Online Feature Store** serving |

### PyFunc vs Plain Flavor
| Situation | Use |
|---|---|
| Single unmodified sklearn/xgboost estimator | Plain flavor (`mlflow.sklearn.log_model`, etc.) |
| Custom preprocessing, custom ensembling logic, non-standard predict() | `mlflow.pyfunc.PythonModel` |
| Native sklearn ensemble (VotingClassifier/StackingClassifier) with no custom logic | Plain sklearn flavor still works |

### MLflow Tracking Feature Mapping
| Tracking Need | MLflow Feature |
|---|---|
| Compare N hyperparameter trials under one experiment | Nested runs (`mlflow.start_run(nested=True)`) |
| Auto-capture params/metrics with minimal code | Autologging |
| Promote/manage model lifecycle | Model Registry (aliases/stages) |
| Store arbitrary files (plots, configs) | `mlflow.log_artifact` |

### Scenario Decomposition Technique
1. Extract the pattern keyword FIRST (independent groups / single model many trials / huge search space + cluster / leakage clue).
2. Match keyword to tool table above BEFORE reading distractors.
3. Time-box: ~60-90 seconds per dense scenario; re-read at most twice.
4. If ambiguous, default to the most 'independent/parallel' cue — it's the most heavily tested pattern.

### Commonly Confused Pairs
- Optuna (tuning one model) vs Ray Tune (tuning at cluster scale) vs pandas Function API (many independent models).
- Delta time travel (table-level rollback) vs point-in-time lookup (row-level as-of join for leakage prevention).
- Plain flavor (standard estimator) vs PyFunc (custom logic required).
