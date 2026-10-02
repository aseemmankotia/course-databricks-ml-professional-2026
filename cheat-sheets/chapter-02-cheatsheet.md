# Chapter 2 Cheat Sheet — Scaling & Hyperparameter Tuning

## pandas Function API
| Method | Requires groupBy | Use Case | Output |
|---|---|---|---|
| `applyInPandas` | Yes | Train one model per key (embarrassingly parallel) | One pandas DF per group, matching declared schema |
| `mapInPandas` | No | Partition-level transforms, no grouping semantics | Iterator of pandas DFs per partition |

Both require an explicit output `schema` (StructType or DDL string) declared up front.

## Optuna (Single-Node HPO)
```python
import optuna
def objective(trial):
    lr = trial.suggest_float('lr', 1e-5, 1e-1, log=True)
    depth = trial.suggest_int('depth', 2, 10)
    ...
    return score
study = optuna.create_study(direction='maximize')  # default sampler: TPESampler
study.optimize(objective, n_trials=100, n_jobs=-1)  # parallel across DRIVER cores only
```
- Default sampler: **TPESampler**. Alternatives: `RandomSampler`, `GridSampler`, `CmaEsSampler`.
- `n_jobs` parallelizes across driver cores — **never** reaches worker nodes.
- Best for: single model, driver-bound, quick iterative search.

## Ray on Databricks (Cluster-Bound)
```python
from ray.util.spark import setup_ray_cluster
setup_ray_cluster(num_worker_nodes=8)
from ray import tune
tuner = tune.Tuner(trainable, param_space=search_space)
results = tuner.fit()
```
- **Ray Tune**: distributes many hyperparameter trials across the whole Ray/Spark cluster.
- **Ray Train**: distributes training of one large model (e.g., across multiple GPU nodes).
- Can wrap Optuna as a search algorithm inside Ray Tune (`OptunaSearch`) for smarter sampling at cluster scale.

## Decision Table: Optuna vs Ray vs pandas Function API
| Scenario | Best Tool |
|---|---|
| Many independent small models (per store/SKU/device) | pandas Function API (`applyInPandas`) |
| Single model, quick driver-only HPO | Optuna |
| HPO needing full-cluster parallel trials | Ray Tune |
| One large model too big/slow for one node | Ray Train |

## Trade-offs: Driver-Bound vs Cluster-Bound
| | Driver-Bound (Optuna default) | Cluster-Bound (Ray Tune/Train) |
|---|---|---|
| Scales with | Driver cores only | All worker nodes |
| Setup complexity | Low | Higher (Ray cluster setup) |
| Best for | Few trials, small models | Many trials or large models |
| Common exam trap | Assuming `n_jobs=-1` distributes across the cluster (it doesn't) | Assuming Ray Tune is needed for simple per-group model training (overkill) |

**Exam notes:**
- `applyInPandas` needs `groupBy` + schema; `mapInPandas` needs only schema, no grouping.
- Optuna trials never leave the driver unless explicitly bridged into a Ray/Spark-distributed executor.
- Hyperopt + SparkTrials is deprecated on current Databricks ML runtimes — Optuna and Ray are the tested tools.
