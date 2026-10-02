## Chapter 11: Deployment Strategies and Serving Patterns: Blue-Green, Canary, and Shadow — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | Blue-green deployment: traffic switch pattern? | Instant 100% cutover from old (blue) to new (green) version after validation — no gradual ramp. Exam tip: 'validated then switched entirely/immediately' = blue-green. |
| 2 | Canary deployment: traffic pattern? | Gradual percentage-based shift (e.g., 5% → 25% → 100%) of REAL user traffic to the new model version. Exam tip: canary users DO receive new-model responses, unlike shadow. |
| 3 | Shadow deployment: does the new model affect user-facing responses? | No. Traffic is mirrored/copied to the new model for comparison; only the original model's output is returned to users. Exam tip: shadow = zero user-facing risk. |
| 4 | Which deployment strategy has the highest blast radius if the new model is bad? | Blue-green — 100% of traffic switches instantly, so a bad model impacts all users until rolled back. Canary limits blast radius; shadow has none. |
| 5 | Databricks Model Serving endpoint config field for splitting traffic between model versions? | traffic_config.routes[].traffic_percentage per served entity — used to implement canary rollouts. |
| 6 | Best serving pattern for scoring 50M rows nightly with no latency requirement? | Batch inference using mlflow.pyfunc.spark_udf on a scheduled Spark job — cheaper than keeping a real-time endpoint always on. |
| 7 | Best serving pattern for an unbounded Kafka stream needing results within seconds? | Streaming inference: Structured Streaming applying the model via mlflow.pyfunc.spark_udf, typically writing to Delta via foreachBatch. |
| 8 | Best serving pattern for a synchronous, low-latency user-facing prediction (e.g., checkout fraud check)? | Real-time Model Serving endpoint queried via REST at /serving-endpoints/{name}/invocations. |
| 9 | Key metrics to monitor on a Model Serving endpoint? | Latency (p50/p90/p99), QPS, error rate (4xx/5xx), and provisioned resource utilization/scaling behavior (including scale-to-zero cold starts). |
| 10 | Symptom: rising p99 latency + rising error rate while QPS stays flat — first thing to check? | Check if the endpoint is scaling from zero / under-provisioned (cold start) — a common cause of correlated latency+error spikes without a traffic increase. |
| 11 | Can a custom PyFunc model registered in Unity Catalog be served directly by a Model Serving endpoint? | Yes — PyFunc is a natively supported flavor; register to catalog.schema.model_name, then reference the UC model version as a served entity, no format conversion needed. |
| 12 | Shadow vs canary — which is used purely for pre-production validation with zero user impact? | Shadow deployment. Canary always exposes a real (smaller) subset of users to the new model's actual output. |
