## Chapter 1: Single-Node vs Distributed Training: Choosing the Right Approach — Practice Questions

**Q1.** A team has a training dataset that fits comfortably in the memory of a single large VM (50 GB) but needs to run 200 hyperparameter combinations for an XGBoost model as fast as possible on a Databricks cluster with 8 worker nodes. Which approach BEST meets this requirement?

A) Convert the dataset to a Spark DataFrame and use a distributed Spark XGBoost estimator to spread the training across all 8 workers
B) Use Hyperopt with SparkTrials to distribute the 200 single-node XGBoost training runs across the cluster's workers in parallel
C) Use scikit-learn's GridSearchCV with n_jobs=-1 on the driver node only
D) Convert the dataset to a Pandas API on Spark DataFrame and manually loop through the hyperparameter combinations

<details>
<summary>Answer</summary>

**Correct: B**

Since the data fits in memory (data-parallel training is unnecessary), the bottleneck is the 200 independent trials. Hyperopt's SparkTrials distributes each single-node trial to a different worker, using cluster parallelism for the search itself.

**Why the others are wrong:**
- Distributed Spark XGBoost training is meant for data too large for one node; adds unnecessary overhead when 50GB fits in memory and doesn't parallelize the 200 trials.
- n_jobs=-1 only uses cores on the driver, ignoring the other 7 worker nodes and wasting cluster resources.
- Pandas API on Spark does not parallelize hyperparameter search across the cluster; manual looping runs sequentially on the driver.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A dataset of 2 TB of clickstream data stored in Delta Lake will not fit into the memory of even the largest available single-node VM. The team must train a logistic regression model on the FULL dataset. Which approach BEST meets this requirement?

A) Sample 10% of the data down to a size that fits in memory, then train with scikit-learn's LogisticRegression
B) Use Pandas API on Spark to load the full dataset, then call to_pandas() and train with scikit-learn
C) Provision a single-node cluster with maximum available memory and use XGBoost's external-memory mode
D) Use pyspark.ml.classification.LogisticRegression, which implements a distributed, data-parallel training algorithm

<details>
<summary>Answer</summary>

**Correct: D**

Spark ML's LogisticRegression is a data-parallel distributed algorithm — it trains across all executors without requiring the full dataset to fit on one node, satisfying the 'full dataset' requirement.

**Why the others are wrong:**
- Sampling discards data and violates the requirement to train on the full 2TB dataset.
- to_pandas() collects the entire dataset onto the driver as a single pandas object, causing an out-of-memory failure for 2TB of data.
- A single VM still cannot hold 2TB in memory; external-memory mode adds complexity and slow disk I/O and is not the standard Databricks pattern.

</details>

---

**Q3.** Raw data of 300 GB requires extensive joins and aggregations before it's reduced to a small 2 GB feature table for training a scikit-learn model. Which approach BEST meets this requirement?

A) Perform all feature engineering using native pandas on a single node by downsampling the 300GB dataset first
B) Perform feature engineering by calling scikit-learn Pipeline transformers directly on the Spark DataFrame
C) Perform feature engineering with Pandas API on Spark distributed across the cluster, then convert the final small feature table to pandas for scikit-learn
D) Convert the entire 300GB dataset to a single pandas DataFrame first, then filter down to the needed features

<details>
<summary>Answer</summary>

**Correct: C**

Pandas API on Spark (pyspark.pandas) lets you scale pandas-style transformations across the cluster for the large raw dataset, then convert only the small resulting feature table to native pandas for the single-node model — matching data size to the right tool at each stage.

**Why the others are wrong:**
- Downsampling 300GB to fit a single node loses data before feature engineering is even applied, reducing accuracy.
- scikit-learn transformers operate on pandas/NumPy arrays, not Spark DataFrames, and will fail or require an unsupported conversion.
- Converting all 300GB to pandas first will exceed driver memory and cause an OOM failure.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** You need to join a large distributed sales fact table with a small (50 MB) dimension table without triggering a shuffle. Which mechanism should you use?

A) repartition() on the dimension table to spread it evenly across executors
B) Pandas API on Spark's persist() method on the dimension table
C) Delta Lake's OPTIMIZE ZORDER BY command on the dimension table
D) A broadcast join, using the broadcast() hint or automatic broadcasting below the configured threshold

<details>
<summary>Answer</summary>

**Correct: D**

Broadcast joins copy the small table to every executor's memory, allowing the join to happen locally without shuffling the large table across the network.

**Why the others are wrong:**
- repartition() redistributes data but still requires a shuffle for the join; it does not avoid the shuffle.
- persist() caches data in memory/disk but does not change join strategy or avoid a shuffle.
- ZORDER optimizes file layout for query pruning on Delta tables; it does not control join execution strategy.

</details>

---

**Q5.** A dataset of 200 GB requires distributed, data-parallel gradient-boosted tree training that scales natively across a Spark cluster. Which library should you use?

A) pyspark.ml.classification.GBTClassifier
B) scikit-learn's GradientBoostingClassifier
C) XGBoost's single-node XGBClassifier with early stopping
D) LightGBM's single-machine training API

<details>
<summary>Answer</summary>

**Correct: A**

GBTClassifier is part of Spark MLlib and implements data-parallel training that scales across the cluster, appropriate for datasets too large to fit on a single node.

**Why the others are wrong:**
- scikit-learn's GradientBoostingClassifier is single-node only and cannot process data that doesn't fit in one machine's memory.
- Single-node XGBoost requires the data to fit on one node and does not natively distribute across a Spark cluster.
- LightGBM's single-machine API is single-node only and has the same memory constraint as the other single-node options.

</details>

---

**Q6.** In the Spark ML Pipeline API, which method call actually produces a trained Model object from an Estimator?

A) fit()
B) transform()
C) fit_transform()
D) predict()

<details>
<summary>Answer</summary>

**Correct: A**

fit() is the defining method of an Estimator; calling it on training data returns a fitted Model, which is itself a Transformer.

**Why the others are wrong:**
- transform() is called on a Transformer (including an already-fitted Model) to produce output; it does not train anything.
- fit_transform() is a scikit-learn convenience method and does not exist in the Spark ML API — a common trap for students used to scikit-learn.
- predict() is not a standard Spark ML method; predictions come from calling transform() and reading the output column.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** Which cluster configuration is BEST suited for training a single-node scikit-learn model on a dataset that fits within 64GB of RAM, when distributed Spark processing is not needed?

A) A cluster with 1 driver and 8 worker nodes, each with 16GB RAM, using autoscaling
B) A cluster with a small driver and 1 large GPU-accelerated worker node
C) A single-node cluster with a driver instance that has at least 64GB+ RAM and sufficient cores for scikit-learn's n_jobs parallelism
D) A high-concurrency cluster with Photon acceleration enabled

<details>
<summary>Answer</summary>

**Correct: C**

Single-node libraries like scikit-learn never distribute work to Spark worker nodes, so a single-node cluster (driver acting as the only compute) sized for the dataset in memory is the correct and cost-efficient choice.

**Why the others are wrong:**
- The 8 worker nodes would sit idle since scikit-learn training never leaves the driver, wasting cluster cost.
- GPU acceleration provides no benefit for standard CPU-based scikit-learn training and adds unnecessary cost.
- Photon accelerates Spark SQL/DataFrame query execution, which is irrelevant to single-node scikit-learn training.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** What is the key distinction between data-parallel and model-parallel distributed training strategies?

A) Data-parallel splits the model across nodes while model-parallel splits the dataset across nodes
B) Data-parallel splits the dataset across nodes with each node holding a full copy of the model; model-parallel splits the model itself across nodes when it is too large for one machine
C) Data-parallel and model-parallel are two names for the same distributed training approach
D) Data-parallel is only used for deep learning, and model-parallel is only used for classical machine learning

<details>
<summary>Answer</summary>

**Correct: B**

Data-parallel distributes data shards across nodes while each node holds the full model (e.g., Spark ML algorithms); model-parallel is used when the model itself cannot fit on one machine and is split across nodes.

**Why the others are wrong:**
- This reverses the correct definitions of the two strategies.
- The two strategies solve different bottlenecks (data size vs. model size) and are not interchangeable.
- Model-parallel is most common in large deep learning models, but the distinction is based on model size relative to a single machine, not a strict domain restriction.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

