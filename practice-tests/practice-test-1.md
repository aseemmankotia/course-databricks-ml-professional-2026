# Practice Test 1: Databricks ML Professional Certification Prep

- **Questions:** 45
- **Time limit:** 90 minutes
- **Passing score:** 70%%

> Answers are in `answer-key-1.md`. Track results in `score-tracker.md`.

---

**Q1.** A retail analytics team needs to train 50,000 independent forecasting models, one per store, where each store's historical sales data comfortably fits into the memory of a single worker node. The team wants to parallelize this training across the cluster efficiently rather than looping sequentially on the driver. Which approach BEST meets this requirement?

A) Use a Spark MLlib distributed training algorithm to train one large model across all stores
B) Use Ray Train to distribute a single deep learning model across the cluster
C) Loop through stores sequentially on the driver node using pandas and scikit-learn
D) Use the pandas Function API (applyInPandas) with a GroupedData object to train an independent model per store in parallel

---

**Q2.** A data science team has a 500GB dataset that does not fit in the memory of any single node in the cluster. They need to train ONE model on the entire dataset. Which approach BEST fits this requirement?

A) pandas Function API grouping the data by a partition key
B) Optuna with a joblib-based parallel backend
C) Ray Train with a single-node actor
D) A Spark MLlib distributed algorithm (e.g., Spark ML's RandomForestRegressor) that partitions data across the cluster

---

**Q3.** A team is running Optuna hyperparameter optimization with multiple worker processes across a Databricks cluster, and each worker needs to read and write trial results to a shared study so trials aren't duplicated or lost. Which configuration should they use?

A) Leave storage as the default in-memory SQLite study, since Optuna handles distribution automatically
B) Save each trial's results to separate local pickle files on each worker
C) Use SparkTrials from Hyperopt instead, since Optuna does not support distributed storage
D) Configure Optuna with an RDB storage backend (e.g., a MySQL or PostgreSQL connection URL) shared across all worker processes

---

**Q4.** During a hyperparameter tuning sweep tracked in MLflow, what is the primary purpose of using nested runs (a parent run with multiple child runs)?

A) To log each trial as a child run beneath a parent run so results can be organized, filtered, and compared in the MLflow UI
B) To encrypt hyperparameter values before logging them to the tracking server
C) To automatically register the best-performing model to Unity Catalog
D) To distribute each trial across a different Spark worker node

---

**Q5.** A team needs to deploy a model via Databricks Model Serving that requires a custom preprocessing step — converting raw categorical text into embeddings via a call to an external service — before the underlying model can produce predictions. This preprocessing is not natively supported by a standard scikit-learn transformer. How should this be packaged for deployment?

A) Save the preprocessing function and model as separate pickle files and reference them manually in the Serving endpoint
B) Build a Spark ML Pipeline combining a custom Transformer and the model estimator
C) Wrap the preprocessing logic and the trained model in a subclass of mlflow.pyfunc.PythonModel, implementing predict(), and log it with mlflow.pyfunc.log_model()
D) Use only a scikit-learn Pipeline object, since sklearn Pipelines support arbitrary external API calls natively

---

**Q6.** A feature table includes a time_series_column that records when each feature value was last updated. When building a training set with the Feature Engineering client, the team must ensure each training example only uses feature values that existed at or before its corresponding label's timestamp, to prevent data leakage. Which FeatureLookup parameter accomplishes this?

A) lookup_key, since it always enforces chronological ordering
B) timestamp_lookup_key, which joins feature values as of the label's timestamp using the feature table's time_series_column
C) feature_names, which restricts which columns are joined
D) rename_outputs, which controls column naming during the join

---

**Q7.** A data science team must train an independent XGBoost model for each of 5,000 retail stores using historical sales data stored in a single Spark DataFrame with a 'store_id' column. The team wants each model trained on a single node (since each store's data fits in memory) but wants all 5,000 trainings to run in parallel across the Spark cluster, returning a Spark DataFrame of model metadata. Which approach BEST meets this requirement?

A) Use groupBy('store_id').applyInPandas() with a function that trains one model per group and returns a pandas DataFrame of results
B) Convert the entire DataFrame to a pandas DataFrame and use a Python for loop to train each model sequentially
C) Use Spark MLlib's built-in GBTRegressor with a distributed training job per store_id partition
D) Use mlflow.spark.autolog() to automatically parallelize training across store_id groups

---

**Q8.** A team is running an Optuna hyperparameter search for a scikit-learn model and wants to distribute the trials across all workers of a Databricks cluster instead of running them sequentially on the driver. Which approach correctly distributes Optuna trials across the cluster?

A) Optuna cannot be distributed on Spark clusters, so trials must always run sequentially on the driver node
B) Pass the Optuna study object directly to hyperopt.SparkTrials to enable cluster-wide parallelism
C) Wrap the training function in a pandas UDF and call it once per hyperparameter combination
D) Wrap the optuna.study.optimize() call in a joblib.parallel_backend('spark') context using the joblib-spark backend

---

**Q9.** A model requires custom logic at inference time: it must load two separate pickled preprocessing artifacts, apply a business rule to clip predictions to a valid range, and then call an underlying scikit-learn model. Which MLflow approach correctly packages this custom inference logic for deployment?

A) Subclass mlflow.pyfunc.PythonModel, implement load_context() to load the artifacts and predict() to run preprocessing, prediction, and clipping, then log it with mlflow.pyfunc.log_model()
B) Log the scikit-learn model with mlflow.sklearn.log_model() and apply the preprocessing and clipping logic in the Model Serving endpoint's REST client code
C) Store the preprocessing artifacts as MLflow tags and reference them manually after loading the model with mlflow.pyfunc.load_model()
D) Use mlflow.sklearn.autolog() and configure a custom scorer function to apply the clipping logic during logging

---

**Q10.** A team is creating a training set using the Feature Engineering client's FeatureLookup API, joining a labels DataFrame against a feature table that is updated daily. To prevent label leakage by ensuring each label row only joins to feature values available as of that label's timestamp, which configuration is required?

A) Specify the timestamp_lookup_key parameter in the FeatureLookup so the client performs a point-in-time correct join
B) Set the feature table's primary key to include a date column and manually filter the feature table before every training run
C) Enable Delta Lake time travel on the feature table and manually pass a fixed VERSION AS OF to create_training_set()
D) Set online_store=True when creating the feature table so lookups automatically respect timestamps

---

**Q11.** A data science team has a training dataset of 800 GB that does not fit into the memory of any single node in their cluster, and they want to train a classification model that scales natively across the cluster's distributed memory. Which approach is MOST appropriate?

A) Use scikit-learn's RandomForestClassifier with n_jobs=-1 on the driver node to parallelize across all CPU cores
B) Downsample the dataset using .sample() until it fits comfortably on the driver, then train with pandas and scikit-learn
C) Use a Spark MLlib estimator (e.g., pyspark.ml.classification.RandomForestClassifier) so training is distributed across the cluster's memory
D) Use the pandas Function API to apply a scikit-learn training function to the full dataset in a single group

---

**Q12.** A real-time fraud detection model needs a feature called 'distance_from_home', calculated as the distance between the transaction's location (provided only at request time) and the customer's home address (stored in a feature table). This feature cannot be precomputed and stored ahead of time because the transaction location is unknown until the serving request arrives. Which Feature Engineering client capability should be used?

A) Precompute distance_from_home nightly using a batch job and store it in a feature table with a 24-hour refresh schedule
B) Store the transaction location as a streaming feature table updated via Structured Streaming
C) Define an on-demand feature using a Python User-Defined Function (UDF) registered in Unity Catalog, referenced as a FeatureFunction that combines the request-time input with the looked-up home address
D) Add transaction location as an additional primary key column in the customer feature table

---

**Q13.** A retail company needs to train a separate lightweight regression model for each of 5,000 stores using historical sales data stored in a Spark DataFrame. Each store's model can be trained independently and quickly on a single node's worth of data. Which approach BEST scales this workload on Databricks?

A) Use the pandas Function API with applyInPandas, grouping by store, to train and return one model's metrics per group in parallel
B) Use Optuna in single-node mode to tune each store's model sequentially
C) Use MLflow autologging to automatically parallelize training across stores
D) Use the Feature Engineering client's on-demand features to generate per-store models

---

**Q14.** A team wants to run 500 Optuna hyperparameter tuning trials for a gradient boosting model, distributing the trials across all worker nodes of a Databricks cluster instead of running them all on the driver. Which approach should they use?

A) Use Hyperopt's SparkTrials class since Optuna has no distributed support
B) Run Optuna trials only on the driver node since it cannot be parallelized
C) Wrap Optuna's study.optimize call with joblib's Spark backend (joblib-spark) to distribute trials across the cluster
D) Use Ray Tune instead since Optuna cannot integrate with Databricks clusters

---

**Q15.** A data scientist enables mlflow.sklearn.autolog() and then calls GridSearchCV.fit() to tune a RandomForestClassifier across multiple parameter combinations and CV folds. What happens to the run tracking by default?

A) You must manually wrap every fold in mlflow.start_run(nested=True) because autologging does not track cross-validation
B) Nested runs are only supported when using Unity Catalog model registry
C) mlflow.autolog() automatically creates a nested child run for each parameter combination/fold under the parent run, with no manual nested run code required
D) Autologging logs only the final best estimator and discards all intermediate fold-level runs

---

**Q16.** Under what circumstance is it necessary to implement a custom MLflow PyFunc model (mlflow.pyfunc.PythonModel) rather than logging a model with a native MLflow flavor (e.g., mlflow.sklearn, mlflow.xgboost)?

A) Whenever logging a scikit-learn model, since scikit-learn has no native MLflow flavor
B) Only when deploying to Model Serving endpoints, regardless of model complexity
C) Whenever a run needs to be logged as a nested run inside a parent run
D) When the model requires custom preprocessing/postprocessing logic, business rule overrides, or combines multiple underlying models in a way no native MLflow flavor supports

---

**Q17.** A fraud detection team uses the Feature Engineering client's create_training_set() with a feature table that includes a timestamp column, specifying timestamp_lookup_key when joining features to labeled events. What does this configuration primarily ensure?

A) It automatically deduplicates rows containing the same primary key in the feature table before training
B) It ensures each training example is joined only to feature values that existed at or before its label timestamp, preventing future data leakage
C) It converts all batch feature lookups into on-demand Python functions computed at inference time
D) It enables streaming ingestion of features into the offline feature table in real time

---

**Q18.** A company wants to train a large PyTorch deep learning model using distributed data parallelism across GPU worker nodes, and prefers a Python-native distributed computing model rather than Spark's DataFrame-centric parallelism. Which Databricks-supported framework should they use?

A) The pandas Function API, since it is designed for GPU-accelerated deep learning
B) Ray, which supports distributed Python-native workloads such as PyTorch training and integrates with Databricks clusters via the Ray on Spark package
C) Optuna, since it natively distributes PyTorch training loops
D) The Feature Engineering client, which manages distributed GPU training internally

---

**Q19.** A data scientist is using the Databricks Feature Engineering client to build a training set. The label DataFrame contains a 'transaction_ts' column, and features must reflect only the feature values that were valid as of each transaction's timestamp (no future data leakage). Which approach correctly enforces this when calling fe.create_training_set()?

A) Define a FeatureLookup with lookup_key set to the join key and timestamp_lookup_key set to 'transaction_ts', then pass it to fe.create_training_set()
B) Manually filter each feature table's DataFrame to rows with a date column less than the transaction date before joining with the label DataFrame
C) Enable Change Data Feed on the feature table and merge only the latest change records into the label DataFrame
D) Define a FeatureLookup with only lookup_key set to the join key, since create_training_set() automatically infers point-in-time correctness from row order

---

**Q20.** A team on Databricks Runtime ML 15 is migrating a hyperparameter search off Hyperopt with SparkTrials, which Databricks has deprecated. They want to run a single Optuna study whose trials are distributed across the cluster's worker nodes, with results logged to MLflow. Which approach BEST meets this requirement?

A) Run study.optimize() with n_jobs set to the number of worker nodes so Optuna distributes trials automatically
B) Use the pandas Function API with applyInPandas so each Spark partition runs an independent Optuna study
C) Continue using Hyperopt with SparkTrials, since Databricks Runtime ML still fully supports it for distributed search
D) Wrap the Optuna study.optimize() call with the joblib Spark backend (joblibspark) so individual trials are distributed as Spark tasks across the cluster

---

**Q21.** A company registers ML models in Unity Catalog. They want the model serving endpoint to always route production traffic to the current best model version without redeploying the endpoint configuration each time a new version is promoted. Which approach BEST meets this requirement?

A) Set the 'champion' alias on the current best model version and configure the serving endpoint to reference the alias instead of a fixed version number
B) Manually update the endpoint's model version field in the serving configuration every time a new best model is identified
C) Use MLflow Model Registry stages (Production/Staging) since aliases are not supported for models registered in Unity Catalog
D) Deploy a new serving endpoint every time a new model version becomes champion, then update DNS to point to the new endpoint

---

**Q22.** A data science team monitors a production feature table for distribution drift. One feature is customer age (a continuous numeric value) and another is customer segment (categorical). Which statistical test should be used to detect drift specifically in the customer age feature?

A) Kolmogorov-Smirnov (KS) test
B) Chi-square test
C) Jaccard similarity
D) Cohen's kappa

---

**Q23.** You are configuring Lakehouse Monitoring on an inference table that logs batch predictions daily. To compute profile and drift metrics over rolling time windows (e.g., one day at a time), which parameter must you configure in the monitor definition?

A) custom_metrics
B) baseline_table_name
C) slicing_exprs
D) granularities

---

**Q24.** A team wants a production model to automatically retrain when Lakehouse Monitoring detects significant feature drift, without any manual intervention. Which design BEST implements this requirement?

A) Manually review the monitoring dashboard each morning and start the retraining notebook if drift appears high
B) Configure an alert on the drift metric table produced by Lakehouse Monitoring that triggers a Databricks Job, and have that job run the retraining pipeline and register the new model version
C) Schedule the retraining job to run every hour regardless of drift, since Lakehouse Monitoring cannot trigger jobs
D) Set the model serving endpoint to automatically retrain itself whenever request latency increases

---

**Q25.** Which statement correctly differentiates legacy MLflow Model Registry stages from Unity Catalog model aliases?

A) Aliases are fixed named states (e.g., Production, Staging, Archived) that a model version must pass through sequentially, while stages are flexible mutable pointers
B) Stages and aliases are functionally identical; aliases are simply a renamed version of stages in Unity Catalog
C) Aliases only apply to the legacy Workspace Model Registry, while stages are the Unity Catalog equivalent used for governance
D) Stages are a fixed set of named states (e.g., Staging, Production, Archived) applied to a model version, while aliases are flexible, user-defined named pointers that can be assigned to any model version and reassigned at any time

---

**Q26.** A team wants to deploy the same ML training and inference pipeline to dev, staging, and production workspaces using Databricks Asset Bundles, with environment-specific cluster sizes and job schedules. Which construct should they use to define these environment-specific overrides?

A) Create separate Git branches for each environment and manually edit the job JSON before each deployment
B) Use a single databricks.yml file with a top-level 'targets' section defining dev, staging, and prod, each overriding resource properties like cluster config and job schedule
C) Use Databricks CLI environment variables exclusively, since Asset Bundles do not support environment-specific configuration
D) Define entirely separate, independent databricks.yml files for each environment with no shared configuration

---

**Q27.** A team registers a forecasting model in Unity Catalog. They want to promote a specific model version to production use without renaming or copying the model, and they want the model serving endpoint to automatically use whichever version is currently promoted, without any endpoint config changes when a new version is promoted. Which approach should they use?

A) Set the 'Champion' alias on the desired model version and reference that alias in the serving endpoint configuration
B) Copy the promoted model version into a new registered model named 'production_model'
C) Use legacy MLflow Model Registry stage transitions (Staging -> Production) on the Unity Catalog model
D) Add a 'Production' tag to the model version and manually update the serving endpoint each time

---

**Q28.** A company wants to define their ML training and deployment pipeline (jobs, model serving endpoints, and permissions) as version-controlled code in git, and deploy identical infrastructure to dev, staging, and production workspaces using a single CLI command with environment-specific configuration overrides. Which Databricks feature should they use?

A) Delta Live Tables pipelines with per-environment catalogs
B) Databricks Asset Bundles with target-specific configuration overrides
C) Databricks Repos with manual notebook promotion between workspaces
D) MLflow Projects with a conda environment file

---

**Q29.** A production job retrains a forecasting model weekly and logs each run with MLflow. Before promoting a newly retrained model, the pipeline must automatically compare it against the current production model on the same holdout set and only promote the new model if its metric exceeds the production model's metric by at least 2%. Which approach best implements this?

A) In the retraining job, evaluate the current Champion-aliased model version and the new run on the same holdout set, and conditionally reassign the alias only if the improvement threshold is met
B) Manually review the metrics in the MLflow UI each week and update the alias by hand if the new model is better
C) Always reassign the 'Champion' alias to the newest model version immediately after each weekly retraining run
D) Rely on Lakehouse Monitoring drift alerts to decide whether the new model should be promoted

---

**Q30.** A Lakehouse Monitor is configured on an inference table containing both numeric features (e.g., transaction_amount) and categorical features (e.g., merchant_category). Which statistical tests does Lakehouse Monitoring use by default to detect distribution drift between the baseline and current time windows for these two feature types, respectively?

A) Chi-squared test for numeric features, Kolmogorov-Smirnov test for categorical features
B) Kolmogorov-Smirnov test for numeric features, chi-squared test for categorical features
C) T-test for both numeric and categorical features
D) Kolmogorov-Smirnov test for both numeric and categorical features

---

**Q31.** A model serving endpoint has inference tables enabled, continuously writing prediction logs to a Delta table. The ML team wants a Lakehouse Monitor on this table to automatically recompute drift metrics every hour as new inference data arrives, without anyone manually triggering a refresh. Which configuration should they use?

A) Set the monitor's metric granularity to '1 hour' and rely on that setting alone to trigger recomputation
B) Enable Auto Loader on the inference table so it implicitly triggers the monitor refresh
C) Schedule a Databricks Job that triggers an hourly refresh of the Lakehouse Monitor
D) Instruct data scientists to manually refresh the monitor once per hour

---

**Q32.** When building automated tests for an ML training pipeline that will run in a Databricks Job as part of CI/CD, which combination of tests provides the most appropriate coverage before deploying the pipeline to production?

A) Only unit tests that check the model's final evaluation metrics after a full training run
B) Manual notebook execution by a data scientist before merging each pull request
C) Unit tests for individual transformation and feature engineering functions using small synthetic DataFrames, plus an integration test that runs the pipeline end-to-end on a sample dataset and validates the resulting model can be loaded and produces predictions
D) Only integration tests that run the full pipeline on the complete production dataset for every CI run

---

**Q33.** A fraud detection model is served via a Databricks Model Serving endpoint. The ML team enables inference tables and creates a Lakehouse Monitoring InferenceLog monitor to track drift on the continuous feature transaction_amount. They want to automatically trigger a Databricks Job that retrains the model whenever the drift metrics show a statistically significant distribution shift for this feature. Which approach BEST implements this requirement?

A) Schedule the retraining job to run every night using a Job trigger regardless of drift metrics
B) Manually review the monitoring dashboard each morning and kick off retraining if drift looks visually apparent
C) Set the InferenceLog monitor's built-in 'auto_retrain' parameter to true when creating the monitor
D) Create a SQL alert on the monitor's auto-generated drift metrics table that fires when the KS test p-value falls below the chosen threshold, and configure the alert to invoke a webhook that runs a Databricks Job

---

**Q34.** A team wants to define and deploy an end-to-end ML pipeline — including a Databricks Job, an MLflow model training notebook, and a model serving endpoint configuration — as version-controlled infrastructure that can be deployed consistently across dev, staging, and production workspaces through CI/CD. Which Databricks feature should they use?

A) Delta Live Tables pipelines
B) MLflow Projects
C) Unity Catalog Volumes
D) Databricks Asset Bundles

---

**Q35.** A team registers a new candidate model version in a Unity Catalog registered model and wants to route only 10% of production traffic to it while the currently promoted version continues to handle 90%, without changing the name consumers use to reference the served model. Which mechanism enables this canary approach when configuring the serving endpoint?

A) Assign the new version the alias 'Champion' and immediately delete the old version
B) Assign the new version the alias 'Challenger' and configure the serving endpoint to split traffic between the 'Champion' and 'Challenger' aliased versions
C) Move the new version to the 'Staging' stage and the old version to the 'Production' stage
D) Tag the new version with the key 'canary=true' and rely on MLflow to automatically route traffic based on tags

---

**Q36.** An InferenceLog monitor tracks a categorical feature 'payment_method' with values such as 'credit_card', 'paypal', and 'bank_transfer'. Which statistical test does Lakehouse Monitoring apply to detect distribution drift for this categorical feature between the baseline window and the current time window?

A) Chi-squared test
B) Kolmogorov-Smirnov test
C) Pearson correlation coefficient
D) Z-test for a single proportion

---

**Q37.** A company deploys a classification model both as a Databricks Model Serving real-time endpoint and as a nightly batch scoring job that writes predictions to a Delta table. They want a single Lakehouse Monitoring setup that captures drift metrics consistently for both deployment modes. Which requirement must be met so both can be monitored using the InferenceLog monitor type?

A) Only the real-time serving endpoint can be monitored — batch scoring jobs are not supported by Lakehouse Monitoring
B) The batch scoring job must first be converted into a Structured Streaming job before an InferenceLog monitor can be attached
C) Both deployment modes must log to the same MLflow experiment ID for the monitor to unify their metrics
D) Both deployment modes must write their predictions, along with model version identifiers and timestamps, into a Delta table matching the schema an InferenceLog monitor expects

---

**Q38.** In a CI pipeline for a Databricks ML project, a data scientist writes a test that calls the trained model's predict() function on a small synthetic DataFrame and asserts that the output schema and value ranges are correct, without invoking any Databricks Job, feature table, or serving endpoint. What type of test is this?

A) Integration test
B) Load test
C) Unit test
D) Canary test

---

**Q39.** A production model's inference table is monitored using Databricks Lakehouse Monitoring. The team wants to detect distribution drift in two columns: 'region' (a categorical feature with 5 possible values) and 'transaction_amount' (a continuous numerical feature). Which statistical test pairing does Lakehouse Monitoring apply to correctly detect drift for these two column types?

A) Kolmogorov-Smirnov test for 'region' and chi-square test for 'transaction_amount'
B) Chi-square test for 'region' and Kolmogorov-Smirnov test for 'transaction_amount'
C) Kolmogorov-Smirnov test for both columns, since it is a universal distribution test
D) Chi-square test for both columns, since Lakehouse Monitoring only evaluates categorical drift

---

**Q40.** A team wants to deploy an identical ML retraining job definition to dev, staging, and production workspaces using one Databricks Asset Bundle, but each environment needs a different cluster size, job schedule, and notebook path parameter. Which Databricks Asset Bundle mechanism should they use to achieve this without duplicating the job definition?

A) Create three separate databricks.yml files with fully independent resource definitions for each environment
B) Use the 'resources' block alone and hardcode environment-specific values directly into each job task
C) Define a single 'targets' section in databricks.yml with per-target overrides for variables, cluster configuration, and schedule
D) Use MLflow Model Registry aliases to point the same job definition at different environments

---

**Q41.** A company wants to deploy a new fraud-detection model version but must limit risk by exposing it to only a small percentage of live traffic first, gradually increasing exposure while monitoring error rates. Which strategy fits, and how is it implemented on Databricks Model Serving?

A) Canary deployment: configure the serving endpoint's traffic_config to route a small percentage (e.g. 5%) to the new served entity, increasing it over time as metrics stay healthy
B) Shadow deployment: route duplicate traffic to the new model but discard its responses so users never see them
C) Blue-green deployment: immediately cut 100% of traffic over to the new served entity
D) Batch inference: score requests offline nightly and update a predictions Delta table instead of serving live traffic

---

**Q42.** A retailer needs product recommendations returned within 200ms of a webpage load request. Which serving approach should be used?

A) A batch inference job scheduled hourly that writes recommendations to a Delta table for the site to read
B) A real-time Databricks Model Serving REST endpoint that returns predictions synchronously per request
C) A Structured Streaming job that continuously scores incoming events and appends results to a table
D) A manually triggered notebook using mlflow.pyfunc.spark_udf to score a batch of pending requests

---

**Q43.** A data science team needs to score 500 million rows nightly using a registered model and write results to a Delta table for downstream BI dashboards. There is no low-latency requirement. Which approach is most appropriate?

A) Deploy a real-time Model Serving endpoint with autoscaling enabled to handle the scoring
B) Deploy the new model as a shadow deployment behind the existing production endpoint
C) Run batch inference using mlflow.pyfunc.spark_udf inside a scheduled Databricks job
D) Use a Structured Streaming job with a continuous trigger to score the rows

---

**Q44.** A Model Serving endpoint already has two served entities named 'model-v1' and 'model-v2'. You want to canary test model-v2 by sending it 10% of traffic while model-v1 receives the remaining 90%. Which traffic_config block correctly implements this split?

A) {"routes": [{"served_model_name": "model-v1", "traffic_percentage": 10}, {"served_model_name": "model-v2", "traffic_percentage": 90}]}
B) {"served_entities": [{"entity_name": "model-v1", "traffic_percentage": 90}]}
C) {"routes": [{"served_model_name": "model-v1", "traffic_percentage": 90}, {"served_model_name": "model-v2", "traffic_percentage": 10}]}
D) {"routes": [{"served_model_name": "model-v1", "traffic_percentage": 50}, {"served_model_name": "model-v2", "traffic_percentage": 50}]}

---

**Q45.** A custom PyFunc model wrapping a scikit-learn classifier with custom pre/post-processing logic has been logged with mlflow.pyfunc.log_model. Which model name format is required to register this model into Unity Catalog?

A) my_model
B) catalog.schema.my_model
C) workspace.my_model
D) models:/my_model/1

---

