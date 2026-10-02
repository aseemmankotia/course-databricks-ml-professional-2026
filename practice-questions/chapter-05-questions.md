## Chapter 5: Model Development Scenarios: Choosing the Right Approach Under Exam Pressure — Practice Questions

**Q1.** A data science team needs to train a separate demand forecasting model for each of 500 stores independently. Each model is small and trains in under a second, but with 500 stores the total training time on a single node is prohibitive. Which approach BEST fits?

A) Use the pandas Function API (applyInPandas) to train one model per store in parallel across the Spark cluster
B) Use Ray Tune to distribute hyperparameter search for a single global model
C) Use Optuna with a single study to tune all 500 stores simultaneously
D) Use Spark MLlib's built-in distributed training algorithms

<details>
<summary>Answer</summary>

**Correct: A**

The 'independent groups' pattern (one model per store) maps directly to the pandas Function API, which parallelizes many small, independent single-node trainings across the cluster.

**Why the others are wrong:**
- Ray Tune distributes hyperparameter search or large-scale trials, not group-partitioned independent training
- Optuna tunes a single model's hyperparameters; it does not partition work by group
- Spark MLlib distributes the training computation of ONE model, not 500 independent models

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A team is tuning a single XGBoost model, running roughly 200 trials that each complete quickly on one node. They want efficient sampling (e.g., TPE) to minimize trial count, with each trial logged as a nested MLflow run. Which tool should they use?

A) Ray Tune
B) pandas Function API
C) Spark MLlib CrossValidator
D) Optuna

<details>
<summary>Answer</summary>

**Correct: D**

'Single model, many trials, efficient sampling' is the exact keyword pattern for Optuna, the current Databricks-recommended tuning library.

**Why the others are wrong:**
- Ray Tune is for search spaces too large for a single node needing cluster-level distribution — overkill here
- pandas Function API is for training independent models per group, not hyperparameter tuning
- CrossValidator performs grid/random search, not TPE-style efficient sampling

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A deep learning team needs to search a hyperparameter space with over 10,000 combinations, distributed across a 20-node GPU cluster, with early stopping to kill underperforming trials mid-training. Which tool is BEST?

A) Optuna running in a single Python process
B) pandas Function API
C) Ray Tune
D) Hyperopt with SparkTrials

<details>
<summary>Answer</summary>

**Correct: C**

'Huge search space + cluster' plus early-stopping schedulers is the signature Ray Tune scenario for distributed trial execution.

**Why the others are wrong:**
- A single Python process cannot distribute across a 20-node cluster
- pandas Function API is for per-group independent training, not hyperparameter search
- Hyperopt is deprecated on current Databricks ML Runtimes and lacks the same distributed early-stopping support

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A feature table must join historical transaction features to training labels using timestamps so that only data available at the time of prediction is used. Which Feature Store capability implements this?

A) Feature Store point-in-time lookup
B) Online Feature Store serving
C) Delta Lake time travel
D) mlflow.log_metric with a step parameter

<details>
<summary>Answer</summary>

**Correct: A**

Point-in-time lookup performs a row-wise as-of join based on each label's timestamp, preventing future data leakage.

**Why the others are wrong:**
- Online serving is for low-latency inference lookups, not training-time joins
- Delta time travel restores an entire table to a prior version, not row-wise as-of joins
- log_metric with step is unrelated to feature joins

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** Which statement correctly distinguishes distributed data preparation from distributed model training in Spark?

A) Spark DataFrame transformations distribute training a single model; MLlib distributes only data prep
B) Both are performed exclusively using the pandas Function API
C) Distributed data preparation always requires Ray, and distributed training always requires Optuna
D) Spark DataFrame/SQL transformations distribute data preparation across the cluster; Spark MLlib algorithms distribute the actual model training computation

<details>
<summary>Answer</summary>

**Correct: D**

DataFrame/SQL ops parallelize transformations on data; MLlib algorithms parallelize the actual training computation of a single model.

**Why the others are wrong:**
- Reverses the roles of DataFrame transforms and MLlib
- The pandas Function API is only one option among several, not exclusive
- Ray and Optuna are tuning/scaling tools, not required for basic Spark data prep or MLlib training

</details>

---

**Q6.** A model requires custom preprocessing logic, combines predictions from three sklearn models via a custom weighted average, and must expose a single predict() call for downstream serving. Which MLflow approach is required?

A) mlflow.sklearn.log_model with a VotingClassifier
B) mlflow.spark.log_model
C) mlflow.pyfunc.PythonModel wrapping the custom logic
D) mlflow autologging with no custom wrapper

<details>
<summary>Answer</summary>

**Correct: C**

Custom preprocessing plus a non-standard weighting scheme cannot be represented by a native sklearn estimator, requiring a custom PyFunc wrapper.

**Why the others are wrong:**
- VotingClassifier only supports built-in voting/averaging schemes, not arbitrary custom preprocessing or weighting logic
- There is no Spark ML model involved here
- Autologging captures standard training calls, not custom ensemble/preprocessing logic

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** You are logging 50 hyperparameter trials for a single model using Optuna and want each trial visible as a separate row under one parent experiment run for comparison in the MLflow UI. Which MLflow feature should you use?

A) Nested runs (mlflow.start_run(nested=True))
B) Model Registry stages
C) Feature Store logging
D) mlflow.log_artifact for each trial

<details>
<summary>Answer</summary>

**Correct: A**

Nested runs let each trial log as a child run under a parent run, enabling side-by-side comparison in the MLflow UI.

**Why the others are wrong:**
- Registry stages track model lifecycle status, not trial-level comparison
- Feature Store logging is unrelated to trial tracking
- log_artifact stores files but doesn't create the parent/child run comparison structure

</details>

---

**Q8.** A scenario states: 'A retail company trains one model per product category (12 categories), each model uses a small dataset that fits in a single node's memory, and hyperparameter tuning for each model needs only a handful of trials.' Applying the scenario decomposition technique, which keyword should you extract FIRST to choose the correct tool?

A) 'small dataset' -> choose Spark MLlib
B) 'hyperparameter tuning' -> choose Ray Tune
C) 'single node's memory' -> choose Optuna directly with no parallelization
D) 'independent groups' (one model per category) -> choose pandas Function API

<details>
<summary>Answer</summary>

**Correct: D**

The dominant tested pattern is 'independent groups,' which maps directly to the pandas Function API regardless of dataset size or trial count.

**Why the others are wrong:**
- Dataset size does not drive tool choice here; MLlib distributes a single model's training, the wrong pattern
- Ray Tune is reserved for large search spaces needing cluster distribution, not a handful of trials
- Ignoring the group-parallelization opportunity wastes the primary cluster benefit the scenario is testing

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

