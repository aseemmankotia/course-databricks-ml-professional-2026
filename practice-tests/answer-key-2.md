# Answer Key — Practice Test 2

**Q1. Correct: A**

applyInPandas (the pandas Function API) is designed exactly for this pattern: many independent, single-node-sized model-training tasks that need to run in parallel across a Spark cluster. Grouping by store ID sends each group's pandas DataFrame to a worker task, where a normal single-node scikit-learn model is trained and results are returned as rows in a Spark DataFrame.

---

**Q2. Correct: A**

FeatureLookup's timestamp_lookup_key parameter tells create_training_set which column in the label DataFrame to use for an as-of join, ensuring only feature values with a timestamp at or before the label's event_timestamp are retrieved, which is the built-in mechanism for point-in-time correctness in the Feature Engineering client.

---

**Q3. Correct: A**

Optuna integrates with joblib, and by registering the joblib-spark backend, calls to study.optimize(..., n_jobs=-1) dispatch trials as Spark tasks that run in parallel across the cluster's executors, which is the supported way to distribute Optuna trials on Databricks.

---

**Q4. Correct: A**

Nested runs in MLflow require an active parent run context; calling mlflow.start_run(nested=True) inside the objective function while the parent run is still open creates a proper child run for each trial, linking it under the parent via parentRunId automatically.

---

**Q5. Correct: D**

For custom PyFunc models, the logged signature is the mechanism MLflow and Model Serving use to validate that incoming JSON/pandas payloads match the expected column names and types, catching malformed requests before they reach potentially fragile custom preprocessing code.

---

**Q6. Correct: D**

On-demand features are computed at inference/scoring time from raw inputs supplied in the request, which is required here because the feature depends on the exact request timestamp and cannot be materialized in advance in any feature table.

---

**Q7. Correct: D**

applyInPandas() (the pandas Function API) lets you group data by a key and run an arbitrary Python function -- including full model training -- on each group's pandas DataFrame in parallel across executors, which is the standard Databricks pattern for many-small-models workloads.

---

**Q8. Correct: C**

Databricks documents using the joblib-spark backend so that joblib.parallel_backend('spark') registers Spark as the execution engine, and passing n_jobs to study.optimize() causes trials to run in parallel across the cluster's executors.

---

**Q9. Correct: B**

Databricks supports launching a Ray cluster on top of a Spark cluster via setup_ray_cluster(), after which Ray Tune provides advanced schedulers like ASHA and PBT, and its built-in MLflow integration (MLflowLoggerCallback) automatically logs trial results.

---

**Q10. Correct: A**

timestamp_lookup_key tells the Feature Engineering client to perform a point-in-time join, matching each label row to the most recent feature value with a timestamp at or before the label's own timestamp column, which is the mechanism that prevents leakage.

---

**Q11. Correct: C**

A custom mlflow.pyfunc.PythonModel subclass lets you embed arbitrary pre- and post-processing code directly inside predict(), and log it as a self-contained pyfunc model that works identically across batch inference, Model Serving, and any framework, with no framework-specific pipeline constraint.

---

**Q12. Correct: A**

On-demand features are registered as Python functions with the Feature Engineering client so the exact same computation logic is used consistently both for real-time inference (computed on raw request inputs) and for generating historical values used in training, ensuring training/serving consistency.

---

**Q13. Correct: C**

groupBy().applyInPandas() is the canonical pattern for training many independent, small, group-wise models (e.g., per-store or per-SKU) in parallel across the cluster, with each group's data materialized as a local pandas DataFrame for a Python function and a declared output schema.

---

**Q14. Correct: D**

Optuna integrates with joblib via optuna.integration and study.optimize(..., n_jobs=...), and registering the joblib-spark backend lets those joblib-parallelized trials execute across Spark cluster workers.

---

**Q15. Correct: A**

FeatureLookup supports a timestamp_lookup_key parameter that, combined with the entity lookup_key, performs an as-of / point-in-time join so each training row only sees feature values that existed at or before its own event timestamp — this is the built-in mechanism for preventing leakage from time-varying features.

---

**Q16. Correct: A**

The standard PythonModel pattern loads heavy artifacts once in load_context() using context.artifacts (paths resolved relative to the packaged model, portable across environments), then predict() is invoked repeatedly per inference request to apply the scaler and combine outputs — this avoids reloading artifacts on every call and avoids hardcoded absolute paths.

---

**Q17. Correct: A**

Ray on Spark lets you provision a Ray cluster co-located with your Databricks Spark cluster's nodes, and Ray Train provides native support for coordinating gradient synchronization across multiple GPU workers for a single distributed PyTorch training job — this is a supported Databricks pattern for multi-node deep learning.

---

**Q18. Correct: C**

SparkTrials integrates with MLflow tracking by creating a parent run when fmin() is called, and automatically starts a nested child run for each individual trial; with autologging enabled, each child run captures that trial's hyperparameters, metrics, and model artifacts separately, letting you compare trials in the Experiments UI under one parent.

---

**Q19. Correct: A**

The pandas Function API's applyInPandas (or mapInPandas) is purpose-built for the 'many small, independent models' pattern: Spark partitions the DataFrame by the grouping key (store_id), and each partition's data — which is small enough to fit on one node — is handed to a pandas UDF that trains a full scikit-learn model locally. This achieves embarrassingly parallel training across the cluster with minimal overhead.

---

**Q20. Correct: A**

FeatureLookup's `timestamp_lookup_key` parameter tells the Feature Engineering client which column in the label DataFrame to use for matching against the feature table's timestamp column, performing an as-of join that only pulls feature values valid at or before each label's timestamp. This, combined with `lookup_key` for the entity join key, is exactly the mechanism used to guarantee point-in-time correctness and prevent leakage.

---

**Q21. Correct: C**

Unity Catalog model registry uses aliases (mutable pointers to a specific version) instead of the legacy stage model. set_registered_model_alias lets you repoint 'champion' to a new version instantly, and code referencing models:/<name>@champion automatically picks up the new version with zero code changes and zero downtime.

---

**Q22. Correct: D**

Lakehouse Monitoring's drift detection uses the Kolmogorov-Smirnov (KS) test to compare distributions of continuous numeric columns and the chi-square test to compare distributions of categorical columns between the baseline and current windows.

---

**Q23. Correct: B**

Lakehouse Monitoring can compute drift metrics on inference tables and raise alerts when thresholds are breached. Wiring that alert to trigger a Databricks Job run (via the Jobs API or a webhook-based automation) creates an event-driven, drift-triggered retraining pipeline that avoids unnecessary scheduled runs and eliminates manual intervention.

---

**Q24. Correct: B**

Databricks Asset Bundles (DABs) are the source-controlled, YAML-based IaC framework for defining jobs, pipelines, and deployments, with 'targets' that let you override cluster configs, parameters, and alias references per environment (dev/staging/prod) and deploy consistently through CI/CD (e.g., `databricks bundle deploy -t prod`).

---

**Q25. Correct: B**

A unit test isolates a single function or method — here, predict() — from external systems (Spark clusters, databases, serving infrastructure) and checks its output shape/type against known input fixtures, making it fast, deterministic, and appropriate for CI without needing live infrastructure.

---

**Q26. Correct: D**

Unity Catalog does not have a built-in automated validation gate — validation logic must be implemented explicitly. Encoding the validation checks and conditional alias assignment (set_registered_model_alias only on success) inside a Databricks Job, deployed and orchestrated via a Databricks Asset Bundle CI/CD pipeline, creates a repeatable, auditable, code-driven promotion gate that blocks bad models before they reach @Champion.

---

**Q27. Correct: C**

UC model registry uses aliases (not legacy stages) as mutable named pointers to specific versions. Reassigning @Champion is atomic and instantaneous, and because versions are immutable, rollback is just reassigning the alias back to v12.

---

**Q28. Correct: A**

Databricks Asset Bundles (DAB) use YAML-defined resources and support multiple deployment targets (e.g., dev/staging/prod) with different variables, enabling `databricks bundle deploy -t staging` and `-t prod` from the same source-controlled bundle, ideal for CI/CD gating.

---

**Q29. Correct: C**

InferenceLog is the Lakehouse Monitoring profile type specifically designed for model inference/scoring tables — it requires model_id, prediction, and optionally label columns, enabling automatic computation of performance metrics (accuracy, F1) alongside drift statistics.

---

**Q30. Correct: B**

A drift alert should trigger conditional, gated retraining: produce a challenger, evaluate it against the champion on a holdout/validation set, and only promote if it demonstrably improves on the metric that matters — preventing automatic promotion of an unvalidated or worse model.

---

**Q31. Correct: B**

Lakehouse Monitoring applies the chi-squared test to detect distributional drift in categorical features by comparing observed category frequency distributions between baseline and current windows.

---

**Q32. Correct: B**

Fast, cheap unit tests on small representative data validate transformation logic and pipeline wiring (schema, shapes, expected exceptions) without the cost of full-scale training, making them appropriate as a pre-merge CI gate.

---

**Q33. Correct: D**

Unity Catalog does not support the legacy stage concept (None/Staging/Production/Archived). Instead, it uses mutable, named aliases (e.g., "champion", "challenger") that you attach to specific model versions and repoint as needed, which is exactly the pattern for tracking a serving version vs. a candidate under validation.

---

**Q34. Correct: C**

Lakehouse Monitoring's drift metrics table applies the two-sample Kolmogorov-Smirnov test to continuous numerical columns (comparing distribution of baseline vs. current windows) and the chi-squared test to categorical columns, reporting a p-value for each.

---

**Q35. Correct: C**

Databricks Asset Bundles are designed for exactly this case: define resources (jobs, clusters, etc.) once, then use the "targets" section to declare per-environment overrides (cluster size, variables, data paths, permissions) so the same bundle deploys differently to dev/staging/prod without duplicating job logic.

---

**Q36. Correct: B**

Lakehouse Monitoring writes metrics to Delta tables (profile and drift metrics), which can be queried with SQL alerts. A SQL alert condition on the F1 threshold can trigger a webhook/notification action that invokes a Databricks Job (via the Jobs API or a job trigger) to run retraining and candidate evaluation logic, fully automating the retrain-and-evaluate loop.

---

**Q37. Correct: B**

Unit testing the PyFunc wrapper's predict() method directly with representative and edge-case inputs, run automatically in CI before the model is ever logged, catches schema and logic bugs early and cheaply, which is the standard practice for validating custom pyfunc models before they enter the registry.

---

**Q38. Correct: B**

Lakehouse Monitoring uses the same monitor object and profile/drift metric definitions regardless of how the underlying inference table is populated. Metric computation is refresh-based (scheduled or manually triggered), scanning newly arrived data since the last refresh — it is not a true row-by-row real-time computation, whether the table was written to via structured streaming or batch jobs.

---

**Q39. Correct: D**

InferenceLog is the profile type designed for model serving/inference tables (batch or streaming), and it supports granularity windows (e.g., '1 day') for periodic drift computation against a baseline. Lakehouse Monitoring automatically selects the appropriate statistical test per column type: chi-squared for categorical/string columns and Kolmogorov-Smirnov for numeric columns.

---

**Q40. Correct: C**

Asset Bundles let you define a multi-task Job with explicit task dependencies and run_if conditions, enabling a gated pipeline: unit tests must pass, then training/registration occurs, then an integration/validation task compares the candidate against the current Champion, and only on success does a final task programmatically move the Champion alias. This is the supported, automatable pattern for pre-deployment validation with Unity Catalog model aliases.

---

**Q41. Correct: B**

Model Serving natively supports multiple served entities on one endpoint via traffic_config, with a traffic_percentage per entity summing to 100. This is the built-in canary mechanism, and the split can be updated via API/UI without redeploying or changing infrastructure.

---

**Q42. Correct: D**

Databricks Model Serving has no native shadow-traffic feature. True shadow deployment requires application-level orchestration that duplicates each live request to both a production and a shadow endpoint, returns only the production result, and logs the shadow prediction separately for comparison.

---

**Q43. Correct: D**

mlflow.pyfunc.spark_udf() wraps a logged model as a vectorized Spark UDF, enabling distributed, in-cluster batch scoring across all 500M rows using existing Spark compute with no external network calls — the recommended pattern for large-scale nightly batch inference.

---

**Q44. Correct: D**

Cold-start spikes occur because scale-to-zero fully deprovisions compute during idle periods, forcing a cold container/model load on the next request. Disabling scale-to-zero and keeping a minimum of one warm provisioned concurrency unit eliminates the cold start while autoscaling above that baseline still limits cost versus running at max capacity constantly.

---

**Q45. Correct: C**

Unity Catalog model registry references version aliases with the '@alias' syntax appended to the three-level model name: 'models:/catalog.schema.model@alias'. This is the current supported way to load a specific tagged version without hardcoding a version number.

---

