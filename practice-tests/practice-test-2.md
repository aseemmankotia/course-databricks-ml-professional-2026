# Practice Test 2: Databricks ML Professional Certification Prep

- **Questions:** 45
- **Time limit:** 90 minutes
- **Passing score:** 70%%

> Answers are in `answer-key-2.md`. Track results in `score-tracker.md`.

---

**Q1.** A data scientist needs to train 2,000 independent store-level demand forecasting models on a Databricks cluster with 16 worker nodes. Each individual model's training data fits comfortably in the memory of a single executor, and each model trains in under 5 seconds with scikit-learn. The goal is to train all 2,000 models as quickly as possible by using the cluster's parallelism. Which approach is correct?

A) Use the pandas Function API's applyInPandas, grouping the Spark DataFrame by store ID so each group's data and model training run as an independent pandas task distributed across the cluster
B) Use Hyperopt's SparkTrials class to parallelize the training of the 2,000 models across the cluster
C) Collect the full dataset to the driver and train all 2,000 models sequentially with a Python for loop using pandas
D) Use Spark MLlib's distributed regression estimators to fit one distributed model across the entire dataset instead of separate per-store models

---

**Q2.** A team is building a training set with the Feature Engineering client for a churn model. The label table has an event_timestamp column marking when each label was observed, and the feature table is updated continuously in near real time. To avoid label leakage from features that were written after the label event occurred, which configuration correctly enforces a point-in-time correct join when calling create_training_set?

A) Pass a FeatureLookup with the timestamp_lookup_key parameter set to the label table's event_timestamp column so the client performs an as-of join against the feature table's timestamp column
B) Enable Delta Time Travel on the feature table and query it with an AS OF VERSION clause matching the label table's write version
C) Add a version column to the feature table's primary key and pass only the lookup_key argument to FeatureLookup
D) Perform a manual Spark SQL join between the label table and feature table using a window function to select the nearest prior feature timestamp, bypassing the Feature Engineering client

---

**Q3.** A team wants to run Optuna hyperparameter tuning trials in parallel across the worker nodes of a multi-node Databricks cluster, rather than running all trials sequentially on the driver. Which approach correctly achieves distributed trial execution with Optuna on Databricks?

A) Configure Optuna's study.optimize() call to use the joblib-spark backend, which submits each trial as a Spark task and distributes them across the cluster's executors
B) Use Hyperopt's SparkTrials class instead, since Optuna has no mechanism for distributing trials across a Spark cluster
C) Run study.optimize() as normal; Optuna automatically detects the active SparkSession and distributes trials across workers with no additional configuration
D) Use Ray Tune's default local scheduler with Optuna as the sampler on a single-node cluster to achieve parallelism

---

**Q4.** During an Optuna hyperparameter sweep, a data scientist wants every trial's parameters and metrics automatically logged to MLflow as a separate nested child run underneath one parent run representing the entire sweep. Which implementation correctly achieves this nested run structure?

A) Open a parent run with mlflow.start_run(), and inside the Optuna objective function, wrap each trial's logic in mlflow.start_run(nested=True) so each trial creates a nested child run under the active parent
B) Call mlflow.autolog() once before starting the sweep; nested child runs are created automatically for every Optuna trial with no further code required
C) For each trial, manually set the tag 'mlflow.parentRunId' on a new run through the MLflow UI after the sweep has finished
D) Call mlflow.log_params() and mlflow.log_metric() directly inside the objective function without starting any run, since Optuna manages its own internal MLflow run structure

---

**Q5.** A team wraps a scikit-learn classifier together with a custom preprocessing step inside an mlflow.pyfunc.PythonModel subclass and logs it with mlflow.pyfunc.log_model() for deployment to a Model Serving endpoint. Why is explicitly defining a model signature at logging time particularly important in this scenario?

A) The signature automatically generates the conda/pip requirements.txt describing the custom model's Python environment
B) A signature is only required when the model is registered in the legacy Workspace Model Registry and is unnecessary for models registered in Unity Catalog
C) The signature determines whether Model Serving provisions CPU or GPU compute for the endpoint
D) The signature records the expected input and output schema so Model Serving can validate incoming request payloads (column names and types) before invoking the custom predict method

---

**Q6.** A real-time fraud detection model needs a feature representing the number of seconds between the current transaction's request time and the customer's most recent prior transaction. This value depends on the exact moment the scoring request arrives and cannot be known in advance. Which Feature Engineering approach is correct for this requirement?

A) Precompute the feature in a nightly batch job and store it in a feature table, refreshing it once every hour
B) Update the feature via a Structured Streaming job with a 1-second micro-batch trigger interval writing into a streaming feature table
C) Store the feature as a computed column directly in the Unity Catalog table and retrieve it with a low-latency online lookup at serving time
D) Define an on-demand feature using a Python feature function registered with the Feature Engineering client, which computes the value at inference time directly from the raw request payload and the customer's last known transaction time

---

**Q7.** A retail company needs to train 5,000 separate demand-forecasting models, one per store, using a Spark cluster to parallelize training across workers. Each store's data and model fit comfortably on a single node. Which approach correctly distributes this workload across the cluster?

A) Use mlflow.pyfunc.spark_udf() to broadcast a single trained model to all stores
B) Use pandas API on Spark (pyspark.pandas) to train one global model on the concatenated dataset
C) Use Hyperopt with SparkTrials to tune one model architecture across all stores simultaneously
D) Use groupBy('store_id').applyInPandas() so each group's data and model training run as a separate pandas job distributed across executors

---

**Q8.** A data science team wants to run an Optuna hyperparameter study and parallelize the trial execution across all workers of a Databricks cluster, rather than running trials sequentially on the driver. Which configuration correctly achieves this?

A) Pass a SparkTrials object from optuna.integration to study.optimize()
B) Call study.optimize() once per executor using a pandas UDF with mapInPandas, without changing any Optuna configuration
C) Wrap optuna.create_study().optimize() calls inside joblib.parallel_backend('spark') using the joblib-spark integration, and pass n_jobs to optimize()
D) Optuna trials cannot be distributed across a Spark cluster; only Hyperopt supports this pattern

---

**Q9.** A team wants to scale hyperparameter tuning using advanced search algorithms like ASHA and population-based training, running trials across a Ray cluster launched on top of Databricks, while automatically logging each trial's metrics and parameters to MLflow. Which combination of tools satisfies this requirement?

A) Hyperopt with SparkTrials and manual mlflow.log_metric calls inside the objective function
B) setup_ray_cluster() to launch Ray on the Spark cluster, then Ray Tune with an MLflow logger callback for automatic experiment tracking
C) GridSearchCV from scikit-learn distributed via joblib-spark
D) pandas Function API with a full grid search run independently inside each partition

---

**Q10.** A feature table records account balance values with a timestamp column indicating when each value became valid. When building a training set with the Feature Engineering client's create_training_set(), which FeatureLookup parameter must be specified to ensure only feature values that existed at or before each label's event time are joined, preventing label leakage?

A) timestamp_lookup_key
B) feature_names
C) rename_outputs
D) lookup_key

---

**Q11.** A model requires custom tokenization before inference and a custom step converting raw probabilities into business-specific risk categories after inference. This logic must travel with the model artifact so it is applied consistently in every serving environment, regardless of the underlying ML framework used. Which approach correctly implements this?

A) Add the preprocessing library as a pip dependency in the model's conda.yaml and rely on the caller to preprocess inputs manually
B) Store the preprocessing and postprocessing code in a Databricks notebook and link it via model tags for manual reference
C) Subclass mlflow.pyfunc.PythonModel, implement load_context() and predict() to encapsulate pre/post-processing, then log it with mlflow.pyfunc.log_model()
D) Use mlflow.sklearn.log_model(), since scikit-learn Pipeline objects are the only supported way to bundle custom logic with a model

---

**Q12.** A fraud-detection model needs a feature counting transactions in the last 5 minutes, computed from raw streaming transaction events at inference time, while historical values of the same feature must also be available for offline model training. Which Feature Engineering approach correctly satisfies both requirements?

A) Register an on-demand feature function with the Feature Engineering client that computes the feature from raw inputs at inference time, and backfill/compute historical values from the same logic for training
B) Precompute the feature once per day in a batch notebook and never recompute it at inference time
C) Compute the feature synchronously inside the Model Serving endpoint's custom code on every request, without registering it anywhere
D) Store the feature only in a Delta table refreshed hourly by a batch job, and join it at inference time regardless of freshness

---

**Q13.** A retail company needs to train 5,000 independent regression models — one per store — using historical sales data stored in a Spark DataFrame with 50 million rows. Which is the BEST approach on Databricks?

A) Collect the DataFrame to the driver with .toPandas() and loop through stores sequentially with scikit-learn.
B) Use Spark ML's Pipeline API to train a single distributed model across all stores.
C) Use the pandas Function API's groupBy().applyInPandas() to train one model per store in parallel, returning a Spark DataFrame with model metadata per group.
D) Use Optuna's distributed study with the joblib backend to train all models on the driver.

---

**Q14.** Which statement about distributing Optuna hyperparameter tuning trials across a Databricks cluster is correct?

A) Optuna natively distributes trials via a SparkTrials class similar to Hyperopt.
B) Optuna requires converting the search space to a Spark ML ParamGridBuilder before distribution.
C) Optuna cannot be distributed on Spark clusters; it only supports single-node execution.
D) Optuna trials can be distributed across cluster workers using the joblib-spark backend with Optuna's joblib integration.

---

**Q15.** A data scientist is creating a training dataset using the Feature Engineering client's create_training_set() method. The feature table contains a feature that changes over time (e.g., account_balance), and the label DataFrame contains an event_timestamp column for each labeled example. Which configuration correctly ensures point-in-time correctness and prevents label leakage?

A) Specify both lookup_key (entity primary key) and timestamp_lookup_key (matching the label DataFrame's event_timestamp column) in the FeatureLookup so lookups return only feature values valid at or before each label's timestamp.
B) Set lookup_key to the entity primary key only; the Feature Engineering client automatically infers timestamps from the label DataFrame's schema.
C) Manually filter the feature table to remove any rows with timestamps after the maximum label timestamp before calling create_training_set().
D) Use a static feature table snapshot exported at label creation time.

---

**Q16.** A team needs to deploy a custom model that loads a pre-trained sklearn scaler and two XGBoost models from artifacts, applies the scaler to input features, gets predictions from both models, and returns a weighted average. They subclass mlflow.pyfunc.PythonModel. Which implementation approach is correct?

A) Override load_context() to load artifacts via context.artifacts paths, and override predict() to apply the scaler and combine model outputs.
B) Override __init__() to load all artifacts using absolute file paths, and override predict() to perform the weighted average.
C) Override predict() only, loading artifacts directly from the MLflow run URI each time predict() is called.
D) Store the scaler and models as class attributes at class definition time before logging the model with mlflow.pyfunc.log_model().

---

**Q17.** A data science team has a 5 TB feature dataset and wants to train a single deep learning model using PyTorch across multiple GPU nodes on a Databricks cluster. Which approach is most appropriate for coordinating the distributed training?

A) Use Ray on Spark (ray.util.spark) to launch a Ray cluster on top of the Databricks Spark cluster, then use Ray Train to coordinate distributed PyTorch training.
B) Use Spark ML's Pipeline API to wrap the PyTorch model as a Transformer for distributed training.
C) Use the pandas Function API's mapInPandas to shard the dataset and train independent copies of the model on each partition.
D) Use Optuna's distributed study to parallelize model weight updates across workers.

---

**Q18.** When running Hyperopt's fmin() with SparkTrials on Databricks with mlflow.xgboost.autolog() enabled, which statement correctly describes the resulting MLflow run structure?

A) A single flat MLflow run is created containing metrics for all hyperparameter trials averaged together.
B) Each trial creates a completely independent top-level run with no parent-child relationship, requiring manual grouping via run tags.
C) SparkTrials automatically creates a parent run for the fmin() call and a nested child run for each trial, with autologging capturing each trial's parameters and metrics in its respective child run.
D) No MLflow runs are created unless mlflow.start_run() is called manually inside the objective function.

---

**Q19.** A data science team needs to train 5,000 independent scikit-learn regression models, one for each retail store in a chain, using historical sales data. Each store's dataset is small (under 50MB) and fits easily in memory on a single worker node, but the team wants to train all 5,000 models in parallel across their Databricks cluster to minimize wall-clock time. Which approach BEST fits this requirement?

A) Use the pandas Function API (applyInPandas) grouped by store_id to train and return one model's metrics per group in parallel
B) Use Hyperopt with SparkTrials to distribute the training runs across the cluster
C) Stand up a Ray cluster on top of Spark and use Ray Train to distribute training of a single model across all workers
D) Use Spark MLlib's distributed linear regression implementation to train one global model per store

---

**Q20.** A data scientist is using the Feature Engineering client to build a training set. The feature table stores nightly aggregated store metrics along with an `as_of_date` column, and the training labels each have their own event timestamp. The scientist must ensure that, for each label row, only feature values that were available on or before that label's timestamp are joined in, preventing feature leakage from future dates. Which FeatureLookup configuration achieves this when calling `create_training_set()`?

A) Set `timestamp_lookup_key` to the label DataFrame's event timestamp column, in addition to `lookup_key`, so the join matches the most recent feature row at or before that timestamp
B) Set `lookup_key` to the store identifier column only, and rely on `create_training_set()` to automatically exclude future rows
C) Set `feature_names` to explicitly exclude the `as_of_date` column from the returned feature set
D) Set `label_col` to the event timestamp column so the training set is automatically sorted chronologically before the join

---

**Q21.** A retail company manages fraud-detection models in Unity Catalog. They want zero-downtime promotion between a currently serving model and a newly trained candidate, avoiding the deprecated Staging/Production stage model entirely, while allowing serving code to always reference the 'current best' model without redeploying. Which approach BEST manages this Champion/Challenger workflow?

A) Assign the model to different registered model names for each role (e.g., model_champion, model_challenger) and manage them as separate registered models.
B) Use legacy stages (Staging/Production) since Unity Catalog fully supports them for aliasing model versions.
C) Use set_registered_model_alias to assign 'Champion' and 'Challenger' aliases to specific model versions, and reference them via the models:/<name>@champion URI syntax in serving/inference code.
D) Apply custom tags like environment=champion to each model version and filter by tag in application code before loading.

---

**Q22.** You configure a Lakehouse Monitoring inference profile on a table with a continuous numeric feature (transaction_amount) and a categorical feature (payment_method). Which statistical tests does Lakehouse Monitoring apply by default to detect distributional drift on these two columns respectively?

A) Chi-square test for both the continuous and categorical columns.
B) Kolmogorov-Smirnov (KS) test for both columns regardless of data type.
C) Chi-square test for the numeric column and Kolmogorov-Smirnov (KS) test for the categorical column.
D) Kolmogorov-Smirnov (KS) test for the continuous numeric column and chi-square test for the categorical column.

---

**Q23.** A fraud-detection model's performance degrades unpredictably due to data drift rather than on a fixed schedule. The ML team wants retraining triggered automatically only when drift is detected, to minimize unnecessary compute costs. Which architecture BEST meets this requirement?

A) Schedule a Databricks Job to retrain the model nightly regardless of whether drift has occurred.
B) Configure a Lakehouse Monitor on the inference table with a metric-based alert that fires a webhook/API call, which invokes a Databricks Job via the Jobs API to retrain the model.
C) Use a Databricks Job with a file arrival trigger that fires whenever new raw training data lands in a table.
D) Manually retrain the model whenever an on-call engineer notices a problem in a monitoring dashboard.

---

**Q24.** Your ML team needs to deploy the same MLflow pipeline (training job, model registration, and serving endpoint) across dev, staging, and prod workspaces with environment-specific cluster sizes and model alias targets, fully version-controlled and deployable via CI/CD. Which Databricks tool is purpose-built for this?

A) Databricks Repos with manually maintained duplicate notebooks per environment.
B) Databricks Asset Bundles, using target-specific overrides in databricks.yml for cluster sizing and alias targets per environment.
C) MLflow Projects defined with a conda.yaml environment specification.
D) Delta Live Tables pipeline definitions with environment parameters.

---

**Q25.** In a CI pipeline for a Databricks ML project, which test type is MOST appropriate for verifying that a custom PyFunc model's predict() method returns correctly shaped output for a fixed input schema, without invoking Spark or any external data source?

A) An integration test, since it verifies model behavior end-to-end.
B) A unit test, since it isolates the predict() logic against controlled inputs without external dependencies.
C) A load test, since it measures the throughput of repeated predict() calls.
D) A production smoke test executed immediately after deployment.

---

**Q26.** Before promoting a new model version to the @Champion alias in Unity Catalog, the team requires an automated validation job (checking accuracy, fairness metrics, and inference latency) to pass successfully. Which implementation enforces this gate as part of CI/CD?

A) Manually review metrics in the MLflow UI, then manually run set_registered_model_alias if they look acceptable.
B) Rely on Unity Catalog's built-in automatic model validation feature to block alias assignment on failing metrics.
C) Use a model registry webhook that fires after the @Champion alias is assigned to validate the model post-promotion.
D) Configure a Databricks Job that runs the validation checks and calls set_registered_model_alias only if the checks pass, orchestrated as a step in a Databricks Asset Bundle CI/CD pipeline.

---

**Q27.** A production model is registered in Unity Catalog with aliases @Champion (v12) and @Baseline (v8). A newly validated version 15 must become the champion, and the team requires instant rollback capability if a problem is detected in the next 24 hours. Which approach correctly achieves this?

A) Overwrite the tag "role":"champion" on v15 and have the serving endpoint read model tags instead of aliases for routing
B) Create a brand-new registered model named 'model_v15' each time a new champion is promoted, to preserve full independent history
C) Use MlflowClient.set_registered_model_alias() to point @Champion at version 15; keep v12 available so @Champion can be reassigned back to it instantly if needed
D) Use MlflowClient.transition_model_version_stage() to move v15 into the 'Production' stage and v12 out of it

---

**Q28.** A team wants a fully declarative, version-controlled CI/CD process: on every push, validate configuration, run automated tests, then deploy Databricks Jobs, model serving endpoints, and pipelines to a staging target automatically, requiring manual approval before deploying the same configuration to a production target. Which Databricks tool is purpose-built for this workflow?

A) Databricks Asset Bundles with separate 'staging' and 'prod' targets defined in databricks.yml
B) Delta Live Tables pipelines configured with a 'development' flag
C) MLflow Projects packaged with a conda.yaml environment file
D) The Feature Engineering client's feature table lineage graph

---

**Q29.** A batch scoring job writes predictions, a model_id column, and a request timestamp to a Delta table. Ground-truth labels are joined in later. The team wants Lakehouse Monitoring to automatically compute daily accuracy/F1 and compare feature distributions against a baseline. Which monitor profile type should be used when calling create_monitor?

A) Snapshot profile type, since the table is scored in discrete batches rather than continuously
B) TimeSeries profile type, using only the timestamp column for windowing
C) InferenceLog profile type, with problem_type and time granularities configured
D) A custom SQL-based monitor built manually with dbutils, since no built-in profile supports model performance metrics

---

**Q30.** A Lakehouse Monitoring dashboard shows the Kolmogorov-Smirnov test p-value for a key numeric feature has dropped below 0.05 for three consecutive daily monitoring windows, and a SQL alert has fired. What should the automated retraining pipeline do to safely decide whether to promote a new model in response?

A) Immediately retrain nightly regardless of the alert and always reassign the @Champion alias to the newest version
B) On receiving the drift alert, trigger a retraining job to produce a candidate model, run automated offline evaluation against the current @Champion on a holdout set, and only reassign @Champion if the candidate exceeds a defined performance threshold
C) Disable the monitor to reduce alert noise and revisit model performance during the next quarterly review cycle
D) Immediately roll the @Champion alias back to the previous model version without retraining, since drift alerts always indicate the current model has degraded

---

**Q31.** A monitored feature named customer_segment is categorical with five possible values. Which statistical test does Lakehouse Monitoring apply to detect distribution drift for this feature between the baseline window and the current window?

A) Kolmogorov-Smirnov (KS) test
B) Chi-squared test
C) Pearson correlation coefficient test
D) Two-sample Z-test for population means

---

**Q32.** A CI pipeline for an ML repository must gate pull requests that modify feature engineering code, catching breakage before merge, without incurring the cost and time of running full distributed training on the complete production dataset for every PR. Which testing approach is most appropriate to add to the CI pipeline?

A) A manual code review checklist completed by a senior engineer before approving the merge, without executing any code
B) A unit test suite using a small synthetic or sampled dataset that validates individual feature transformation functions and runs a minimal end-to-end training pass to check schema and output shape
C) An integration test that triggers the full production training job against the entire production dataset in the production workspace
D) A post-deployment data quality check configured through Lakehouse Monitoring on the production inference table

---

**Q33.** A team manages ML models in Unity Catalog. They want an identifier that always points to the currently serving version, and a separate identifier for a candidate version being validated before promotion. Which approach should they use?

A) Create two separate registered models named "model_prod" and "model_staging" and always log new versions to whichever is currently active
B) Set the "Production" and "Staging" stages on the model versions, since stages control serving behavior in Unity Catalog
C) Use MLflow Model Registry stages (None/Staging/Production/Archived), since Unity Catalog directly inherits stage-based transitions from the Workspace Model Registry
D) Assign the alias "champion" to the currently serving version and "challenger" to the candidate version, updating the alias assignment as versions are promoted

---

**Q34.** A Lakehouse Monitoring inference table monitor is configured for a model with both continuous numerical features (e.g., transaction_amount) and categorical features (e.g., merchant_category). Which statistical test does Lakehouse Monitoring apply to each feature type, respectively, to detect drift?

A) Chi-squared test for numerical features and Kolmogorov-Smirnov test for categorical features
B) Population Stability Index for both feature types uniformly, regardless of data type
C) Kolmogorov-Smirnov test for numerical features and chi-squared test for categorical features
D) A single Jensen-Shannon divergence metric applied identically regardless of feature type

---

**Q35.** An ML team uses Databricks Asset Bundles to deploy the same training-and-deployment job to dev, staging, and prod. They want the job to use a small cluster with test data in dev, but a larger cluster with production data in prod, without duplicating the job definition. What should they do?

A) Maintain three completely separate databricks.yml bundle files, one per environment, and manually keep them synchronized
B) Hardcode the production cluster configuration in the job definition and manually edit the cluster in the Jobs UI before each dev run
C) Define the job once in the bundle and use environment-specific overrides in the "targets" section of databricks.yml to override cluster size and variables per environment
D) Use a single "workspace" mapping in databricks.yml and modify the job's notebook code to detect environment name at runtime and switch cluster sizes programmatically via the Jobs API

---

**Q36.** A production model's inference table monitor calculates that the F1 score on labeled data (via a join with a ground-truth table) has dropped below a defined threshold. The team wants a fully automated response: create a new candidate model version and evaluate it against the current champion. Which approach best achieves this on Databricks?

A) Manually review the Lakehouse Monitoring dashboard weekly and retrain when someone notices the drop
B) Configure a SQL alert on the monitor's generated metrics table that, when the F1 degradation condition is met, triggers a webhook or Jobs API call to run a Databricks Job that retrains and evaluates a candidate model before promotion
C) Set the model's production alias to auto-update whenever any new version is logged, regardless of evaluation results
D) Rely on Unity Catalog's built-in automatic retraining feature, which retrains models automatically whenever governance policies detect degradation

---

**Q37.** Before registering a custom PyFunc model that wraps a preprocessing pipeline and a classifier, a team wants to catch schema mismatches and broken preprocessing logic in their CI pipeline, prior to logging the model to MLflow. Which testing approach is most appropriate?

A) Skip automated testing since MLflow's model signature enforcement automatically catches all input schema and logic errors at serving time
B) Write pytest unit tests that instantiate the PyFunc wrapper directly, call its predict method with sample DataFrames covering edge cases, and assert on expected output shape and values, integrated into the CI pipeline before model logging
C) Rely solely on integration tests that deploy the model to a serving endpoint and monitor for HTTP 500 errors during the first day of production traffic
D) Manually run the notebook end-to-end once before each deployment and visually confirm the output looks reasonable

---

**Q38.** A model serving endpoint logs requests and predictions to an inference table. The team enables Lakehouse Monitoring on this table. What is a key difference between how monitoring metrics are computed for streaming versus batch inference tables?

A) Streaming inference tables cannot be monitored by Lakehouse Monitoring; only batch tables support monitor creation
B) Both use the same monitor definition and profile/drift metric computation, but the metric refresh is triggered on a schedule (or manually) that re-scans new data in the table, regardless of whether that data arrived via streaming or batch writes
C) For streaming inference tables, monitoring metrics are computed continuously in real time as each row arrives, entirely bypassing scheduled refreshes
D) Batch inference tables require metrics to be computed manually via custom SQL, while streaming tables use only built-in dashboards with no configurable refresh

---

**Q39.** A model's serving payloads are captured in a Unity Catalog inference table that is continuously updated by a Structured Streaming job writing scored requests in near real time. The ML team needs Lakehouse Monitoring to flag drift on a categorical feature, 'region_code', by comparing each day's traffic against a fixed baseline window. Which monitor configuration correctly supports this requirement?

A) Create a TimeSeries profile monitor with daily granularity and rely on the Kolmogorov-Smirnov test, since it is the default statistical test applied to all monitored columns regardless of data type
B) Create a Snapshot profile monitor and manually re-run it once per day, since Snapshot is the only profile type Lakehouse Monitoring supports for tables populated by Structured Streaming
C) Create an InferenceLog profile monitor, but first one-hot encode region_code into numeric indicator columns so the KS test can be applied, since chi-squared tests are not available in Lakehouse Monitoring
D) Create an InferenceLog profile monitor with daily granularity windows; Lakehouse Monitoring automatically applies the chi-squared test to categorical columns like region_code and the KS test to numeric columns

---

**Q40.** An automated retraining pipeline registers a new model version to Unity Catalog after every successful training job. Before promoting a version to the 'Champion' alias for production serving, the team requires: (1) unit tests on the feature engineering code, (2) an integration test scoring the new version against a held-out validation set and comparing it to the current Champion's metrics, and (3) alias promotion only if the new version beats Champion by a defined margin. Which implementation best satisfies these requirements using Databricks Asset Bundles and Jobs?

A) Define a single Job task that trains the model and immediately sets the Champion alias to the new version; use Lakehouse Monitoring alerts to detect and roll back if the new version underperforms after deployment
B) Configure an MLflow Model Registry webhook on the model that automatically promotes any version scoring above a hardcoded R2 threshold, independent of the current Champion's performance
C) Define a multi-task Databricks Job in an Asset Bundle: one task runs pytest unit tests on the feature pipeline, one trains and registers the candidate version, one runs an integration test comparing candidate vs. current Champion metrics, and a final task uses run_if conditions to move the Champion alias only when the comparison passes
D) After each training run, have an engineer manually inspect the MLflow Experiment UI and update the alias through the Catalog Explorer if the metrics look acceptable

---

**Q41.** A team is rolling out a new fraud-detection model version behind a Databricks Model Serving endpoint that currently serves v3 at 100% traffic. Company policy requires that no more than 5% of live traffic reach any unvalidated model version, with the ability to increase that percentage only after 24 hours of clean metrics. Which endpoint configuration correctly implements this canary rollout?

A) Create a second endpoint for v4, and use a load balancer outside Databricks to route 5% of client traffic to the new endpoint
B) Update the serving endpoint's traffic_config to define two served entities (v3, v4) on the same endpoint, setting traffic_percentage to 95 for v3 and 5 for v4, then update the split later
C) Deploy v4 as the only served entity on the endpoint with traffic_percentage 100, and rely on client-side retry logic to fall back to v3 on error
D) Set the endpoint's workload_size to Small for v3 and Large for v4 so v4 naturally receives less traffic due to lower throughput capacity

---

**Q42.** An ML team wants to validate a new recommendation model against live production traffic before it influences any user-facing decision. Every production request should also be scored by the new model, with its predictions logged for offline comparison, but the new model's output must never be returned to the client. How should this be implemented on Databricks?

A) Configure the endpoint's traffic_config to route 0% of traffic to the new model version; Databricks automatically shadow-scores all requests against 0%-weighted entities
B) Enable the endpoint's built-in shadow_traffic flag on the new served entity, which duplicates all inbound requests without returning shadow responses to clients
C) Use Lakehouse Monitoring's drift detection job to automatically replay production inference table logs against the new model version nightly, and treat that as shadow traffic
D) Deploy the new model on a separate serving endpoint and add custom logic upstream (e.g., in the client application or a routing layer) that sends each request to both endpoints, returns only the production endpoint's response, and asynchronously logs the shadow endpoint's predictions

---

**Q43.** A Delta table with 500 million rows must be scored nightly by a registered MLflow pyfunc model. There is no real-time latency requirement, and the team wants to reuse Spark's distributed compute already provisioned for other ETL jobs in the same workflow. Which approach BEST meets these requirements?

A) Deploy the model behind a Databricks Model Serving real-time endpoint and call it row-by-row from a Spark job using the REST API
B) Convert the pyfunc model into a Structured Streaming stateful transformation and run it continuously with a fixed trigger interval
C) Use Feature Serving to publish the 500 million rows to an online store, then query each row individually through the Feature Serving REST endpoint
D) Load the model with mlflow.pyfunc.spark_udf() and apply it as a column expression over the Delta table in a Spark batch job

---

**Q44.** A Model Serving endpoint hosting a mid-size XGBoost model shows P99 latency spikes of several seconds whenever traffic resumes after idle periods, even though sustained-traffic P99 latency is well within SLA. Cost constraints prevent running the endpoint at full capacity 24/7. Which endpoint configuration change BEST addresses the cold-start latency spikes while still controlling cost?

A) Increase the endpoint's workload_size from Small to Large so more compute is always available
B) Enable scale_to_zero and reduce the autoscaling idle timeout so the endpoint scales down faster between bursts
C) Switch the served entity's compute type from CPU to GPU to reduce per-request inference time
D) Disable scale-to-zero and configure a minimum of one always-on provisioned concurrency unit, allowing the endpoint to autoscale up from that warm baseline during bursts

---

**Q45.** A custom PyFunc model was registered in Unity Catalog under catalog 'ml_prod', schema 'fraud', and model name 'txn_scorer'. The model version currently tagged with the alias 'champion' must be loaded for batch scoring using mlflow.pyfunc.load_model(). Which model URI correctly loads that version?

A) models:/ml_prod.fraud.txn_scorer/Production
B) dbfs:/models/ml_prod.fraud.txn_scorer@champion
C) models:/ml_prod.fraud.txn_scorer@champion
D) models:/ml_prod.fraud.txn_scorer/champion

---

