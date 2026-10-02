# Answer Key — Practice Test 1

**Q1. Correct: D**

The pandas Function API lets you group a Spark DataFrame by a key (e.g., store_id) and apply an arbitrary pandas/scikit-learn function to each group in parallel across executors, which is the standard pattern for many-small-models workloads that don't need distributed algorithms.

---

**Q2. Correct: D**

Spark MLlib algorithms are designed to train a single model whose computation is distributed across the cluster by partitioning the data itself, which is required when the dataset exceeds single-node memory.

---

**Q3. Correct: D**

To run distributed Optuna trials across multiple processes or machines, the study must use a persistent shared storage backend (an RDB URL such as MySQL or PostgreSQL) so all workers can read/write the same study state and avoid duplicate or lost trials.

---

**Q4. Correct: A**

Nested runs create a parent-child hierarchy in the MLflow tracking UI so that each hyperparameter trial appears as a child run under a single parent, making it easy to visually group, sort, and compare metrics/params across the whole sweep.

---

**Q5. Correct: C**

Subclassing mlflow.pyfunc.PythonModel lets you bundle arbitrary custom logic (including external API calls) alongside the model into a single self-contained artifact with a defined predict() method, which is exactly what Model Serving loads and invokes — this is the standard pattern for custom inference logic.

---

**Q6. Correct: B**

Specifying timestamp_lookup_key in a FeatureLookup tells create_training_set to perform a point-in-time join against the feature table's time_series_column, returning only the feature values valid as of each label's timestamp, thereby preventing future data from leaking into training.

---

**Q7. Correct: A**

groupBy().applyInPandas() (the pandas Function API) partitions the Spark DataFrame by the grouping column and applies an arbitrary pandas function to each group's data on a worker node in parallel, which is exactly the 'many models' pattern for training thousands of small, independent single-node models at scale.

---

**Q8. Correct: D**

The joblib-spark package registers a 'spark' backend for joblib; running optuna.study.optimize(..., n_jobs=-1) inside a joblib.parallel_backend('spark') context distributes individual Optuna trials across the executors of the Databricks cluster.

---

**Q9. Correct: A**

mlflow.pyfunc.PythonModel is the designed mechanism for wrapping arbitrary custom logic (multiple artifacts, preprocessing, business rules) into a single deployable model. load_context() loads artifacts once at load time, and predict() encapsulates the full custom inference pipeline so it deploys identically everywhere MLflow models are served.

---

**Q10. Correct: A**

The timestamp_lookup_key parameter on FeatureLookup tells the Feature Engineering client which column in the labels DataFrame to use for a point-in-time join, so each label row is matched only to feature values that existed at or before that timestamp, preventing future data (label leakage) from being included in training.

---

**Q11. Correct: C**

Spark MLlib estimators are designed to train a single model whose data and computation are distributed across the cluster, which is required when the dataset (800 GB) exceeds any single node's memory capacity.

---

**Q12. Correct: C**

On-demand features are computed at inference time using a registered Python UDF (a FeatureFunction) that takes request-time inputs (like transaction location) combined with pre-computed feature values (like home address from a feature table) — this is exactly the mechanism for features that cannot be precomputed because they depend on data only known at serving time.

---

**Q13. Correct: A**

The pandas Function API (applyInPandas/mapInPandas) lets you partition a Spark DataFrame by a grouping key and run an arbitrary single-node Python function (e.g., training a scikit-learn model) on each group in parallel across the cluster, which is exactly suited to 'many small independent models' workloads.

---

**Q14. Correct: C**

Databricks supports distributing Optuna trials by registering the joblib-spark backend and using joblib.parallel_backend('spark') around study.optimize(), which pushes trial execution out to the Spark workers.

---

**Q15. Correct: C**

MLflow's scikit-learn autologging integrates with GridSearchCV/RandomizedSearchCV and automatically logs each parameter combination as a nested run beneath a parent run, along with the best estimator's params and metrics on the parent, without requiring manual mlflow.start_run(nested=True) calls.

---

**Q16. Correct: D**

Custom PyFunc wrappers are needed when inference requires logic beyond a single library's predict() call, such as custom feature transforms, ensembling multiple models, applying business rules, or returning non-standard output formats, none of which native flavors handle out of the box.

---

**Q17. Correct: B**

Specifying timestamp_lookup_key enforces point-in-time correctness: for each training row, only feature values with a timestamp at or before the label's timestamp are joined in, which is essential for avoiding leakage of future information into training data.

---

**Q18. Correct: B**

Ray is supported on Databricks (via the Ray on Spark integration) and provides a general-purpose, Python-native distributed computing framework well suited to distributed deep learning training such as PyTorch DDP, task/actor-based parallelism, and Ray Train, independent of Spark's DataFrame model.

---

**Q19. Correct: A**

The Feature Engineering client enforces point-in-time correctness natively when a time-series feature table is used: setting timestamp_lookup_key on the FeatureLookup tells create_training_set() to join each label row only with feature values whose timestamp is at or before the label's timestamp, preventing leakage without any manual filtering code.

---

**Q20. Correct: D**

joblibspark provides a Spark-backed joblib parallel backend; registering it and wrapping study.optimize() in a joblib.parallel_backend('spark') context lets Optuna dispatch individual trials as Spark tasks across worker nodes, giving true cluster-wide distributed tuning for a single-node model while still logging each trial to MLflow.

---

**Q21. Correct: A**

Unity Catalog model aliases are mutable named pointers (e.g., 'champion') that can be reassigned to any version instantly via API/UI. If the serving endpoint config references the alias, moving the alias to a new version automatically changes what the endpoint serves with no endpoint redeploy or downtime.

---

**Q22. Correct: A**

The Kolmogorov-Smirnov (KS) test compares the cumulative distributions of a continuous numeric variable between a baseline window and a current window, making it the standard test Lakehouse Monitoring uses for numeric feature drift.

---

**Q23. Correct: D**

The 'granularities' parameter defines the time-bucket size (e.g., '1 day', '1 hour') used to aggregate the inference table into windows for computing profile metrics and drift statistics over time.

---

**Q24. Correct: B**

Lakehouse Monitoring writes drift metrics to metric tables that can back a SQL alert; that alert can notify or trigger automation (e.g., a webhook that kicks off a Databricks Job) that runs the retraining pipeline and registers the new model version — a fully closed-loop, automated design.

---

**Q25. Correct: D**

Legacy MLflow stages were a fixed enumerated set (None/Staging/Production/Archived) directly assigned to a version. Unity Catalog replaces this with aliases — arbitrary, user-defined string labels (e.g., 'champion', 'challenger') that can point to any version and be moved freely, giving more flexible, multi-alias workflows.

---

**Q26. Correct: B**

Databricks Asset Bundles support a 'targets' block in databricks.yml where each target (e.g., dev, staging, prod) can override resource definitions such as cluster configuration, job schedules, and parameters, while sharing the common pipeline definition — this is the standard CI/CD pattern for bundles.

---

**Q27. Correct: A**

Unity Catalog model lifecycle management uses aliases (e.g., Champion, Challenger) instead of legacy stages. Referencing the alias in the serving endpoint means the endpoint automatically resolves to whichever version currently holds that alias, with no endpoint changes required when the alias is reassigned.

---

**Q28. Correct: B**

Databricks Asset Bundles (DABs) let teams define jobs, serving endpoints, and permissions as YAML-based infrastructure-as-code, version it in git, and deploy to multiple named targets (dev/staging/prod) via 'databricks bundle deploy' with target-specific overrides.

---

**Q29. Correct: A**

Programmatically loading the current Champion model, scoring both models on the same holdout data, and conditionally reassigning the alias via the MLflow/Unity Catalog client implements automated, threshold-based model selection as part of the retraining job.

---

**Q30. Correct: B**

Lakehouse Monitoring computes the Kolmogorov-Smirnov (KS) test for numeric feature distributions and the chi-squared test for categorical feature distributions when comparing a baseline window to a current window.

---

**Q31. Correct: C**

Lakehouse Monitoring requires an explicit refresh trigger to recompute metrics on new data; scheduling a Databricks Job to call the monitor refresh on an hourly cadence automates this without manual intervention.

---

**Q32. Correct: C**

Combining fast unit tests on individual pipeline components with a lightweight end-to-end integration test on a sample dataset validates both correctness of logic and overall pipeline functionality without the cost and latency of running on the full production dataset every CI run.

---

**Q33. Correct: D**

Lakehouse Monitoring writes results to auto-generated profile and drift metrics tables. A Databricks SQL alert can query the drift metrics table and, on threshold breach, trigger a webhook/notification that fires off a retraining Job — this is the standard drift-driven automated retraining pattern.

---

**Q34. Correct: D**

Databricks Asset Bundles (DABs) let you declare jobs, pipelines, and ML resources as YAML in a databricks.yml file, with named deployment targets (dev/staging/prod), making them the purpose-built tool for CI/CD deployment of ML pipelines across workspaces.

---

**Q35. Correct: B**

Unity Catalog models use aliases (e.g., champion, challenger) as mutable pointers to specific versions. Model Serving endpoints can be configured to reference multiple aliased versions and split inbound traffic between them, which is exactly how canary/A-B rollouts are implemented in UC-based model lifecycle management.

---

**Q36. Correct: A**

Lakehouse Monitoring uses the chi-squared test to compare categorical feature distributions between baseline and current windows, recording the resulting statistic and p-value in the auto-generated drift metrics table.

---

**Q37. Correct: D**

An InferenceLog monitor operates on a Delta table containing prediction outputs, model version identifiers, and timestamps in the required schema — this can be populated either automatically via Model Serving inference tables or manually by a batch job, as long as the table schema matches what the monitor expects.

---

**Q38. Correct: C**

This test exercises a single isolated function (predict()) with controlled synthetic input and checks its output in isolation from the rest of the pipeline — the defining characteristic of a unit test.

---

**Q39. Correct: B**

Lakehouse Monitoring applies the chi-square test to detect distributional changes in categorical/discrete columns and the Kolmogorov-Smirnov (KS) test to detect distributional changes in continuous numerical columns, comparing a baseline window to the current window.

---

**Q40. Correct: C**

Databricks Asset Bundles use the 'targets' section in databricks.yml to define environment-specific overrides (workspace, variables, cluster specs, schedules) while sharing a single set of resource definitions, enabling consistent CI/CD promotion across dev, staging, and prod.

---

**Q41. Correct: A**

Canary deployment on Databricks Model Serving is implemented by setting traffic_config percentages across served entities and incrementally shifting the split toward the new version while monitoring latency/error rate, which directly matches the gradual, risk-limited exposure requirement.

---

**Q42. Correct: B**

Sub-second, per-request latency requirements are the defining use case for real-time Model Serving, which hosts the model behind a low-latency REST endpoint and returns predictions synchronously for each request.

---

**Q43. Correct: C**

Large-scale, latency-insensitive scoring is the canonical use case for batch inference: wrapping the registered model with mlflow.pyfunc.spark_udf lets Spark distribute scoring across the cluster within a scheduled job, which is efficient and cost-effective for 500M rows.

---

**Q44. Correct: C**

Traffic splitting on a Databricks Model Serving endpoint is configured under traffic_config.routes, listing each served_model_name with its traffic_percentage; giving model-v1 90 and model-v2 10 matches the required 90/10 canary split, and the percentages sum to 100.

---

**Q45. Correct: B**

Unity Catalog requires models to be registered under the three-level namespace catalog.schema.model_name, so mlflow.register_model (or the log_model registered_model_name parameter) must use that fully qualified format for the model to be tracked and governed in UC.

---

