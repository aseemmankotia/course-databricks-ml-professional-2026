## Single-Node vs Distributed Training

### Decision Table
| Scenario | Data fits on 1 node? | Approach |
|---|---|---|
| Small/medium dataset | Yes | scikit-learn, XGBoost (single-node) |
| Large dataset, need full-data training | No | Spark ML (data-parallel) |
| Small data, many hyperparameter trials | Yes | Hyperopt + **SparkTrials** (parallelize trials, not data) |
| Spark ML model, hyperparameter search | N/A (data already distributed) | Hyperopt + **Trials** (not SparkTrials) |
| Model too large for 1 machine | N/A | Model-parallel (rare in classical ML; common in deep learning) |

### Data-Parallel vs Model-Parallel
| Type | What's split | Example |
|---|---|---|
| Data-parallel | Dataset across nodes, full model on each | Spark ML LogisticRegression, GBTClassifier |
| Model-parallel | Model itself across nodes | Large deep learning models |

### Spark ML Pipeline API
| Component | Key method | Notes |
|---|---|---|
| Estimator | `.fit(df)` → returns Model | e.g. LogisticRegression, VectorAssembler is a Transformer not Estimator |
| Transformer | `.transform(df)` | Fitted Models are Transformers |
| Pipeline | `.fit(df)` runs stages in order | Produces a `PipelineModel` |
| Param | Named hyperparameter | Used with `ParamGridBuilder` for CV/tuning |
| ⚠️ Trap | No `fit_transform()` in Spark ML | scikit-learn muscle memory error |

### Broadcasting
| Setting | Value |
|---|---|
| `spark.sql.autoBroadcastJoinThreshold` default | 10 MB |
| Force broadcast | `df.join(broadcast(small_df), 'key')` |
| Use case | Small lookup/dimension table joined to large fact table — avoids shuffle |

### Pandas API on Spark
- Use `pyspark.pandas` for pandas-style syntax that scales across the cluster (feature engineering on large raw data).
- Convert final small result with `.to_pandas()` before handing off to scikit-learn.
- ⚠️ `toPandas()` / `to_pandas()` on a large DataFrame collects to the driver — risk of OOM.

### Cluster Sizing Guide
| Workload | Cluster type |
|---|---|
| Single-node scikit-learn/XGBoost training | Single-node cluster, driver-only, sized for dataset in RAM |
| SparkTrials hyperparameter search on single-node model | Multi-worker cluster (1 trial per core/worker) |
| Spark ML distributed training | Multi-worker cluster sized for data volume + shuffle needs |
| GPU-accelerated deep learning | GPU-enabled instance types |

### Common Exam Traps
- Using a big multi-worker cluster for single-node libraries → wasted idle workers.
- Using SparkTrials with an already-distributed Spark ML model → wrong tool.
- Calling `.to_pandas()`/`toPandas()` on data too large for driver memory → OOM.
- Confusing `fit_transform()` (scikit-learn) with Spark ML's separate `.fit()`/`.transform()`.
