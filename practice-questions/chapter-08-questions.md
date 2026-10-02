## Chapter 8: Testing ML Pipelines: Unit and Integration Tests for Production ML — Practice Questions

**Q1.** A data scientist writes a Spark-based feature transform that computes a ratio of two columns. The team wants unit tests that run in under a second on every commit, without provisioning a cluster. What is the BEST approach?

A) Attach to a running interactive cluster and execute pytest against it for every commit
B) Refactor the ratio logic into a pure Python/Pandas function tested with small in-memory data, and keep the thin Spark wrapper for integration tests
C) Use dbutils.notebook.run against a 1% sample of the production Delta table
D) Skip unit tests entirely and rely only on integration tests, since Spark logic can't be unit tested

<details>
<summary>Answer</summary>

**Correct: B**

Isolating business logic from Spark I/O lets pytest run pure-Python assertions instantly with no cluster; the Spark wrapper is thin enough to be covered later by an integration test.

**Why the others are wrong:**
- Attaching to a cluster adds startup latency and cost to every commit, defeating fast feedback
- Running against a live Delta sample still depends on cluster/job infrastructure and is not a true isolated unit test
- Spark logic IS unit-testable via local SparkSession or by extracting pure functions; skipping unit tests reduces coverage and violates the testing pyramid

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** CI needs to test a Spark-dependent transformation stage without provisioning a Databricks cluster. What should the pytest fixture use?

A) dbx execute against a job cluster for every pull request
B) Serialize the DataFrame to JSON and test with plain Python only
C) A local SparkSession (master('local[*]')) with small in-memory DataFrames
D) MagicMock the entire SparkSession object and assert only that methods were called

<details>
<summary>Answer</summary>

**Correct: C**

A local SparkSession runs in-process with real Spark execution semantics, giving fast, cluster-free CI feedback while still exercising actual DataFrame operations.

**Why the others are wrong:**
- Job clusters are slow to start and cost money for every PR, unsuitable for fast unit-test feedback loops
- Serializing to JSON loses Spark execution semantics for logic that depends on Spark transformations (joins, window functions, etc.)
- Mocking the SparkSession only verifies call patterns, not correctness of the actual transformation output

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A team requires that a candidate model's F1 score exceed the current production (champion) model's F1 by at least 0.02 before promotion via alias. Which implementation correctly enforces this as an automated quality gate?

A) Manually review a metrics dashboard and promote if the numbers look good
B) Enable a model registry webhook on TRANSITION_REQUEST_CREATED that auto-approves every request
C) Add a Delta Live Tables expectation that checks the metric threshold
D) A CI/CD job step loads both models' logged metrics via the MLflow API, compares them programmatically, and only calls set_registered_model_alias if the threshold is met

<details>
<summary>Answer</summary>

**Correct: D**

Programmatic metric comparison in an automated job step enforces the quality bar consistently and is the standard way to gate promotion in CI/CD for ML.

**Why the others are wrong:**
- Manual review is subjective, not repeatable, and not part of an automated pipeline
- Auto-approving all requests defeats the purpose of a quality gate — it promotes regardless of the metric threshold
- DLT expectations validate data quality in ingestion pipelines, not model evaluation metrics stored in MLflow

</details>

---

**Q4.** You wrap a scikit-learn classifier in an MLflow PyFunc model. Which pytest assertion BEST validates predict() before production deployment?

A) assert model.predict(input_df) is not None
B) assert len(model.predict(input_df)) > 0
C) assert model.predict(input_df).dtype == "float64"
D) output = model.predict(input_df); assert isinstance(output, pd.DataFrame) and output.shape[0] == input_df.shape[0] and "prediction" in output.columns

<details>
<summary>Answer</summary>

**Correct: D**

This checks output type, shape parity with the input row count, and the expected output column — directly matching the exam's 'predict() output shape/type' objective.

**Why the others are wrong:**
- Checking for non-None is too weak; it would pass even if predict() returned malformed or empty output
- Checking length > 0 doesn't confirm the output row count matches the input, missing shape validation
- PyFunc predict() typically returns a DataFrame, which has no .dtype attribute and would raise an AttributeError; it also ignores shape and column checks

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** A pipeline needs schema, null, and range validation on training data before calling .fit(). Which approach is idiomatic for a testable Databricks ML pipeline?

A) Rely exclusively on Delta Live Tables expectations, even for an ad hoc Pandas training script
B) Wrap .fit() in a broad try/except that catches all exceptions
C) Write pytest-testable validation functions that check df.schema, count nulls per required column, and verify value ranges, failing fast with a clear error before .fit()
D) Manually eyeball a data profile in a notebook before each training run

<details>
<summary>Answer</summary>

**Correct: C**

Explicit, testable validation functions covering schema/null/range checks give clear, fast-failing diagnostics and can be unit tested independently, matching the exam's 'data validation checks' objective.

**Why the others are wrong:**
- DLT expectations are designed for streaming/batch ingestion pipelines, not universally applicable to arbitrary training scripts
- A generic try/except swallows errors without meaningful validation of schema, nulls, or ranges
- Manual notebook inspection isn't automated or repeatable, and can't run in CI

</details>

---

**Q6.** A training pipeline is defined as a Databricks Asset Bundle job. What is the correct way to validate it before promoting to production?

A) Deploy directly to the prod target and monitor for failures
B) Run `databricks bundle deploy -t staging` then `databricks bundle run -t staging <job-key>` to execute the pipeline end-to-end against staging resources first
C) Edit the prod target's variables in databricks.yml to point to a smaller cluster and treat that as staging
D) Run the job manually from the Databricks UI without using the bundle CLI

<details>
<summary>Answer</summary>

**Correct: B**

Asset Bundles support isolated targets (dev/staging/prod); deploying and running against staging validates the full job/task definitions in an environment separate from prod before promotion.

**Why the others are wrong:**
- Deploying straight to prod skips validation and risks breaking production resources
- Repurposing the prod target definition breaks target isolation and doesn't test the actual staging configuration that will later be promoted
- Running manually from the UI bypasses the bundle's YAML-defined resources, so the deployment definition itself is never validated

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** Which statement correctly differentiates unit tests from integration tests for an ML pipeline?

A) Unit tests always require a running cluster while integration tests never do
B) Integration tests make unit tests unnecessary once the pipeline passes
C) Unit tests only apply to Python code and integration tests only apply to SQL code
D) Unit tests validate individual functions (e.g., a feature transform) in isolation; integration tests run a scaled-down version of the full pipeline end-to-end and verify expected artifacts

<details>
<summary>Answer</summary>

**Correct: D**

This matches the standard testing pyramid definition used throughout the MLOps testing objectives: unit tests isolate logic, integration tests exercise the whole pipeline in a real (scaled-down) job.

**Why the others are wrong:**
- This is reversed — unit tests should avoid clusters for speed; integration tests typically DO run in a real scaled-down job
- Unit and integration tests are complementary, not substitutes for each other
- This is a false dichotomy not supported by exam objectives; both test types apply regardless of language

</details>

---

**Q8.** A team wants CI to verify that a training job (ingest → transform → train → register) works end-to-end before merging to main, without waiting on full production-scale data. What is the BEST integration test design?

A) Trigger the job with the full production dataset on every merge to catch all possible issues
B) Only run unit tests on the transform functions and skip integration testing to save cluster costs
C) Manually run the pipeline in the workspace UI once a week
D) Trigger the same job definition via CI with a small sample dataset override (parameter/widget), then assert on job run status and expected artifacts (e.g., registered model version, logged metrics)

<details>
<summary>Answer</summary>

**Correct: D**

Parameterizing the same job/DAG with a small sample dataset keeps real Spark, cluster, and MLflow integration under test while bounding cost/time, then verifying success and artifacts — exactly the 'scaled-down pipeline end-to-end' pattern the exam tests.

**Why the others are wrong:**
- Full production-scale runs on every merge are slow and costly, defeating fast CI feedback
- Skipping integration tests misses failures in wiring between stages (e.g., schema mismatches between transform and train steps)
- A weekly manual run is not automated and provides no CI gate on pull requests

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

