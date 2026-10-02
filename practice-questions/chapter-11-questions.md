## Chapter 11: Deployment Strategies and Serving Patterns: Blue-Green, Canary, and Shadow — Practice Questions

**Q1.** A team wants to test a new fraud-detection model against production traffic without affecting any customer-facing decisions or responses, so they can compare predictions offline before promoting the new model. Which deployment strategy BEST meets this requirement?

A) Shadow deployment mirroring production traffic to the new model
B) Canary deployment routing 10% of traffic to the new model
C) Blue-green deployment with instant cutover
D) Batch inference scoring on a nightly schedule

<details>
<summary>Answer</summary>

**Correct: A**

Shadow deployment sends a mirrored copy of live traffic to the new model for comparison while the old model's output remains the only one returned to users — zero user-facing impact.

**Why the others are wrong:**
- Canary exposes a real subset of users to the new model's actual output, violating the 'no user-facing impact' requirement
- Blue-green instantly switches all traffic, exposing 100% of users to the new model
- Nightly batch scoring does not compare against real-time live traffic as it arrives

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** An ML team must score 50 million rows nightly against a registered Unity Catalog model, writing results to a Delta table for downstream BI. Individual prediction latency is not a concern. Which serving pattern is MOST cost-effective?

A) Batch inference using mlflow.pyfunc.spark_udf in a scheduled Spark job
B) Real-time Model Serving endpoint with autoscaling
C) Shadow deployment against a real-time endpoint
D) Structured Streaming job reading from Kafka

<details>
<summary>Answer</summary>

**Correct: A**

Batch inference with spark_udf on a scheduled Spark cluster avoids the cost of an always-on endpoint and is optimized for large one-time offline scoring jobs.

**Why the others are wrong:**
- A real-time endpoint incurs continuous compute cost even when idle, wasteful for a one-time nightly job
- Shadow deployment is for comparing model versions on live traffic, not for offline scoring workloads
- Structured Streaming from Kafka implies continuous unbounded data, not a scheduled nightly batch

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** You want to shift traffic gradually from model version 3 (currently 100%) to version 4, starting at 10% and increasing weekly on a Databricks Model Serving endpoint. Which configuration field controls this split?

A) scale_to_zero_enabled
B) workload_size
C) served_entities[].entity_version
D) traffic_config.routes[].traffic_percentage

<details>
<summary>Answer</summary>

**Correct: D**

traffic_config.routes[] with traffic_percentage per served entity is the mechanism used to implement gradual canary traffic splitting.

**Why the others are wrong:**
- scale_to_zero_enabled controls autoscaling replicas to zero, unrelated to traffic routing
- workload_size sets the compute size (Small/Medium/Large) of a served entity, not traffic distribution
- entity_version only identifies which model version is served, it does not control percentage of traffic

</details>

---

**Q4.** Which of the following BEST describes a blue-green deployment on a Databricks Model Serving endpoint?

A) Traffic is split 50/50 between two model versions indefinitely
B) A fully validated new ('green') version is deployed, then traffic is switched instantly from 100% old ('blue') to 100% new
C) The new model receives a mirrored copy of production traffic while the old model continues serving all real responses
D) Traffic percentage to the new model is increased incrementally over several days

<details>
<summary>Answer</summary>

**Correct: B**

Blue-green is defined by an instant, complete cutover of traffic from the old to the new fully-validated version.

**Why the others are wrong:**
- A permanent 50/50 split is not a blue-green pattern, which is a full switch, not a steady split
- This describes shadow deployment, where the new model's output never reaches users
- This describes canary deployment, which is gradual rather than instant

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** What is the primary distinction between canary deployment and shadow deployment?

A) Canary requires Unity Catalog while shadow does not
B) Shadow deployment only works with batch inference; canary only works with real-time serving
C) In canary, a portion of real user traffic receives responses from the new model; in shadow, the new model processes copied traffic but its output never reaches users
D) Canary is used for streaming pipelines; shadow is used only for real-time endpoints

<details>
<summary>Answer</summary>

**Correct: C**

The defining difference is whether the new model's output is actually delivered to end users (canary) or only used for internal comparison (shadow).

**Why the others are wrong:**
- There is no Unity Catalog registration requirement difference between the two strategies
- Shadow deployment is typically applied to real-time or mirrored traffic, not exclusively batch
- Both strategies are most commonly discussed in the context of real-time endpoints, not tied exclusively to one or the other by definition

</details>

---

**Q6.** A production Model Serving endpoint shows a rising 5xx error rate and p99 latency increasing from 80ms to 900ms over the last hour, while QPS remains stable. What should you check FIRST?

A) Whether the traffic_config was recently changed to route more traffic to a canary model
B) Whether the registered model was deleted from Unity Catalog
C) Whether the endpoint is scaling from zero (or under-provisioned), causing cold-start delays and timeouts
D) Whether the Databricks workspace region matches the model registry region

<details>
<summary>Answer</summary>

**Correct: C**

Rising latency and error rate with stable QPS is a classic symptom of insufficient provisioned capacity or cold starts from scale-to-zero, which the exam tests as a resource utilization monitoring scenario.

**Why the others are wrong:**
- A traffic_config change alone would not cause errors/latency unless the canary model itself is broken, and stable QPS argues against a routing-driven explanation as the first check
- A deleted model would cause immediate total failure of requests, not a gradual latency/error rise
- Region mismatch is not a real Model Serving failure mode tested by the exam

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** A custom PyFunc model with preprocessing and postprocessing logic is registered in Unity Catalog as catalog.schema.fraud_model. What is required to serve this model from a real-time Model Serving endpoint?

A) Create a serving endpoint that references the Unity Catalog model version as a served entity; PyFunc is natively supported
B) Convert the PyFunc model to an MLflow built-in flavor before serving
C) Export the model to ONNX format first
D) Wrap the model in a Structured Streaming job before creating the endpoint

<details>
<summary>Answer</summary>

**Correct: A**

Databricks Model Serving natively supports custom PyFunc models registered in Unity Catalog as served entities with no format conversion required.

**Why the others are wrong:**
- PyFunc is already a supported serving flavor, so conversion to another built-in flavor is unnecessary
- ONNX conversion is not required for Model Serving to host a PyFunc model
- A Structured Streaming wrapper is unrelated to and unnecessary for real-time endpoint serving

</details>

---

**Q8.** A team needs near real-time fraud scoring on an unbounded stream of transactions arriving via Kafka, with results available within seconds, reusing the same registered model used by their batch scoring job. Which implementation is MOST appropriate?

A) A real-time Model Serving endpoint queried synchronously over REST for every transaction
B) A Structured Streaming job that applies the model using mlflow.pyfunc.spark_udf, writing results to a Delta table
C) A nightly batch job using spark_udf over the full historical dataset
D) A shadow deployment mirroring Kafka traffic to a staging endpoint

<details>
<summary>Answer</summary>

**Correct: B**

Structured Streaming combined with mlflow.pyfunc.spark_udf is the standard pattern for continuous streaming inference, reusing the same model as batch and delivering sub-minute latency to a Delta sink.

**Why the others are wrong:**
- Per-record synchronous REST calls to a real-time endpoint is not the standard scalable pattern for high-throughput Kafka firehose ingestion in this exam context
- A nightly batch job cannot meet a near-real-time (seconds) latency requirement
- Shadow deployment is for model comparison validation, not a production scoring pipeline for delivering results

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

