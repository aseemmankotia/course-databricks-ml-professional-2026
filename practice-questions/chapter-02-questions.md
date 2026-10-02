## Chapter 2: Scaling and Hyperparameter Tuning: pandas Function API, Optuna, and Ray — Practice Questions

**Q1.** A retail analytics team must train one independent demand-forecasting model for each of 8,000 stores. Each store's model is small and fits easily in a single executor's memory. Which approach best leverages the cluster for this embarrassingly parallel training job?

A) Set up a Ray cluster on the Databricks cluster and use Ray Tune to launch 8,000 trials
B) Group the Spark DataFrame by store_id and use applyInPandas to train and return one model's metrics per group
C) Run an Optuna study with n_jobs=-1 on the driver, one trial per store
D) Collect the full dataset to the driver and loop through stores sequentially with scikit-learn

<details>
<summary>Answer</summary>

**Correct: B**

applyInPandas groups the DataFrame by key (store_id) and executes the pandas function per group across executors in parallel — the canonical pattern for many-small-models embarrassingly parallel training.

**Why the others are wrong:**
- Ray Tune is designed for distributing hyperparameter search trials or large-model training, not simple per-group model fitting; it adds unnecessary cluster orchestration overhead here.
- Optuna runs on the driver by default and is meant for hyperparameter optimization, not for distributing independent per-group model training across the cluster.
- Collecting to the driver and looping sequentially wastes the cluster's executors entirely and does not scale.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A data science team has a single XGBoost model configuration and wants to run 2,000 hyperparameter trials as fast as possible, using every worker node in a 16-node Databricks cluster, not just the driver's cores. Which setup satisfies this requirement?

A) optuna.create_study() followed by study.optimize(objective, n_trials=2000, n_jobs=-1) on the driver
B) Set up a Ray cluster with ray.util.spark.setup_ray_cluster() and run Ray Tune across the cluster
C) mapInPandas to fan out trials across partitions
D) Increase the driver node size and rerun Optuna with more threads

<details>
<summary>Answer</summary>

**Correct: B**

Ray Tune, running on a Ray cluster provisioned across the Spark cluster's workers, distributes trials across multiple nodes — the only option here that is genuinely cluster-bound rather than driver-bound.

**Why the others are wrong:**
- n_jobs=-1 in Optuna only parallelizes across the cores of the driver node; it never reaches the worker nodes.
- mapInPandas operates on DataFrame partitions for data transformation, not on trial objects for hyperparameter search — it is not designed for HPO orchestration.
- A bigger driver still limits execution to a single machine's cores, not the full 16-node cluster.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A data scientist has a single-node scikit-learn model and wants a quick, driver-only hyperparameter search using Bayesian optimization, with no need to scale beyond the driver. Which tool is the appropriate choice?

A) pandas Function API with applyInPandas
B) Ray Train
C) Optuna with the default TPESampler
D) Ray Tune with a multi-node Ray cluster

<details>
<summary>Answer</summary>

**Correct: C**

Optuna is purpose-built for single-node, driver-bound hyperparameter search and uses TPESampler by default for efficient Bayesian-style optimization — exactly matching the stated requirement.

**Why the others are wrong:**
- applyInPandas is for grouped model training across executors, not for tuning a single model's hyperparameters.
- Ray Train distributes training of a large model across multiple nodes, which is unnecessary and adds setup overhead for a single-node model.
- Provisioning a Ray cluster for a driver-only need is overkill and wastes cluster resources.

</details>

---

**Q4.** When you call optuna.create_study() without specifying a sampler, which sampler is used by default?

A) TPESampler
B) RandomSampler
C) GridSampler
D) CmaEsSampler

<details>
<summary>Answer</summary>

**Correct: A**

Optuna defaults to TPESampler (Tree-structured Parzen Estimator), a Bayesian optimization method that models promising regions of the search space based on prior trials.

**Why the others are wrong:**
- RandomSampler exists as an explicit alternative but is not the default.
- GridSampler must be manually configured and requires an exhaustive parameter grid; it is never the default.
- CmaEsSampler is a specialized evolutionary strategy sampler that must be explicitly selected.

</details>

---

**Q5.** You need to train a per-group regression model where each group's pandas function returns a DataFrame with a different number of rows than its input group, and you know the exact output schema in advance. Which pandas Function API method is correct?

A) mapInPandas without any groupBy
B) A standard scalar pandas_udf
C) DataFrame.groupBy(key).applyInPandas(func, schema)
D) DataFrame.foreachPartition with a pandas conversion

<details>
<summary>Answer</summary>

**Correct: C**

applyInPandas is designed for exactly this pattern: group by key, apply a pandas function per group, and return a pandas DataFrame per group with a schema declared up front.

**Why the others are wrong:**
- mapInPandas operates on partitions, not grouped keys, so it cannot guarantee per-group semantics.
- A scalar pandas_udf operates column-wise on Series and cannot change row cardinality per group or return an arbitrary DataFrame.
- foreachPartition is an action with no return value; it cannot produce a resulting DataFrame.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** What is the core structural difference between applyInPandas and mapInPandas?

A) applyInPandas can only be used with Delta tables; mapInPandas works with any DataFrame
B) applyInPandas requires a prior groupBy and operates per group; mapInPandas operates on an iterator of pandas DataFrames representing partitions with no grouping
C) mapInPandas guarantees exactly one output row per input row; applyInPandas does not
D) applyInPandas runs on the driver; mapInPandas runs on executors

<details>
<summary>Answer</summary>

**Correct: B**

applyInPandas requires groupBy(...) first and invokes the function once per group; mapInPandas has no grouping requirement and instead receives an iterator of pandas DataFrames for each partition.

**Why the others are wrong:**
- Neither API is restricted to Delta tables — both work on any Spark DataFrame.
- mapInPandas does not enforce row-count parity; both APIs allow the output row count to differ from input.
- Both APIs execute on executors, not the driver; this is not a distinguishing factor.

</details>

---

**Q7.** A team is fine-tuning a large transformer model whose full training process does not fit in memory or complete in reasonable time on a single GPU node, and they need to distribute the actual training computation (not hyperparameter trials) across multiple GPU worker nodes on Databricks. Which tool should they use?

A) Optuna with n_jobs set to the number of cluster cores
B) pandas Function API with applyInPandas
C) Ray Train
D) Hyperopt with SparkTrials

<details>
<summary>Answer</summary>

**Correct: C**

Ray Train is the Ray component built specifically to distribute the training of a single large model across multiple nodes (including GPUs), as opposed to distributing many independent trials or small models.

**Why the others are wrong:**
- Optuna's n_jobs only parallelizes across driver cores and cannot distribute a single model's training computation across nodes.
- applyInPandas is for training many independent per-group models, not for splitting the training of one large model across the cluster.
- Hyperopt with SparkTrials is deprecated on current Databricks ML runtimes and, even historically, distributed trials rather than a single model's training.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** A cluster has a driver and 8 worker nodes. A team must train 500 independent small forecasting models, one per SKU, each completing in under a second. Which approach best utilizes the full cluster rather than bottlenecking on the driver?

A) Ray Tune, treating each SKU as a separate hyperparameter trial
B) Optuna study.optimize with n_jobs=-1
C) A single Ray Train job with one model configuration
D) pandas Function API using applyInPandas grouped by SKU

<details>
<summary>Answer</summary>

**Correct: D**

applyInPandas naturally distributes per-group (per-SKU) work across the cluster's executors, making it the simplest and most efficient fit for many small, independent models — Ray and Optuna are designed for hyperparameter search or single large-model workflows, not this pattern.

**Why the others are wrong:**
- Using Ray Tune's trial mechanism to represent independent models is a misuse of a hyperparameter search tool and adds unnecessary orchestration complexity.
- Optuna's n_jobs only parallelizes across the driver's cores, so worker nodes remain idle.
- Ray Train distributes training of one large model, not many small independent models.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

