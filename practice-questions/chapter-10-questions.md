## Chapter 10: Alerting and Production Governance for ML Systems — Practice Questions

**Q1.** A company has Lakehouse Monitoring enabled on the inference table of a production model. They want drift beyond a threshold to automatically trigger retraining, followed by evaluation, and only then update the Production alias. Which approach BEST meets this requirement?

A) Schedule a Job to retrain the model every night regardless of drift metrics.
B) Use an MLflow Model Registry 'Comment' webhook to kick off retraining whenever anyone adds a comment.
C) Configure Auto Loader to trigger retraining whenever new files arrive in the inference table's storage location.
D) Create a Databricks SQL alert on the monitor's drift metrics table that triggers a Job via a webhook; the Job retrains, evaluates against the current Champion, and updates the alias only if evaluation passes.

<details>
<summary>Answer</summary>

**Correct: D**

This chains alert -> automated retrain -> validation -> conditional promote, matching the exam's expected escalation pattern and keeping a gate before promotion.

**Why the others are wrong:**
- Nightly unconditional retraining ignores drift signal entirely and wastes compute — doesn't satisfy 'triggered by drift'.
- Comment webhooks fire on manual UI comments, not on data drift, so they don't automate the described response.
- Auto Loader reacts to new files landing, not to statistical drift computed by the monitor.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A compliance team needs a record of exactly who moved a Unity Catalog model version's alias to Production and when. Where does this audit trail live?

A) MLflow experiment run tags on the training run.
B) Unity Catalog audit logs, which capture alias assignments and permission changes tied to each model version.
C) Databricks SQL query history for the monitoring dashboard.
D) Delta Lake time travel history on the inference table.

<details>
<summary>Answer</summary>

**Correct: B**

UC audit logs record governance actions (alias moves, grants, revokes) on registered models, providing the required audit trail.

**Why the others are wrong:**
- Experiment tags describe training runs, not registry/alias governance actions.
- Query history tracks SQL execution, not model version lifecycle events.
- Time travel tracks table data changes, not model registry governance actions.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** An exam scenario states: 'Once a new model version passes automated validation tests, it should be promoted to Production without any human step; if it fails, it must remain unpromoted.' Which implementation BEST satisfies this?

A) A Databricks Job with conditional task logic that runs validation tests, then calls the MLflow Client to set the Production alias only if the tests task succeeds.
B) A human reviews the test results in the MLflow UI and manually moves the alias.
C) A Databricks SQL alert sends a Slack message to a human every time a new version is logged.
D) An autoscaling cluster policy is applied to speed up the training job.

<details>
<summary>Answer</summary>

**Correct: A**

Conditional Job tasks (depends_on with run-if-success logic) can gate the alias update programmatically, satisfying the 'no human step' requirement.

**Why the others are wrong:**
- Manual review reintroduces the human step the scenario explicitly disallows.
- A Slack alert only notifies a human — it does not promote or gate anything automatically.
- Cluster autoscaling affects training speed/cost, not promotion logic.

</details>

---

**Q4.** Which Databricks capability should be used to automatically compute statistical drift between a model's training baseline and current inference data?

A) Lakehouse Monitoring
B) Databricks SQL Alerts
C) Unity Catalog Lineage
D) Delta Live Tables expectations

<details>
<summary>Answer</summary>

**Correct: A**

Lakehouse Monitoring generates profile and drift metrics tables comparing current data distributions against a baseline; this is its core function.

**Why the others are wrong:**
- SQL Alerts only query existing metric tables on a schedule; they don't compute drift themselves.
- Lineage tracks data/table/model dependencies, not statistical distribution shifts.
- DLT expectations enforce row-level data quality constraints, not distributional drift analysis.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** You need drift metrics computed by Lakehouse Monitoring to automatically trigger a retraining Job without manual polling. Which mechanism accomplishes this?

A) A Delta Live Tables pipeline reading the metrics table.
B) A cluster-scoped init script on the training cluster.
C) A Databricks SQL Alert on the monitor's drift metrics table with a webhook destination that calls the Jobs API.
D) Refreshing the MLflow Tracking UI periodically.

<details>
<summary>Answer</summary>

**Correct: C**

SQL Alerts support webhook/notification destinations that can invoke the Jobs API, providing the automated trigger path from metric to retraining run.

**Why the others are wrong:**
- A DLT pipeline transforms/validates data; it isn't designed to invoke external Job runs based on alert conditions.
- Init scripts configure cluster environments at startup; they have no relation to metric-based triggering.
- Manually refreshing a UI is not automation and requires human polling, which the requirement excludes.

</details>

---

**Q6.** You configure a Databricks SQL alert on the drift metrics table with condition `avg(js_distance) > 0.2`, evaluated on an hourly schedule. Which statement about this configuration is correct?

A) The alert evaluates in real time on every write to the monitored table automatically.
B) SQL alerts can only be attached to Delta Live Tables event logs, not to Lakehouse Monitoring output tables.
C) The alert re-runs its underlying query on the defined schedule and fires only when the condition is true at that evaluation, not continuously.
D) SQL alerts cannot be attached to derived metrics tables, only to raw inference tables.

<details>
<summary>Answer</summary>

**Correct: C**

Databricks SQL Alerts run their query on a defined schedule (e.g., hourly) and evaluate the condition against the result at that point; they are not event-driven on every write.

**Why the others are wrong:**
- Alerts are schedule-driven, not write-triggered — this is a common misconception tested on the exam.
- SQL alerts can be attached to any queryable table, including DLT event logs, but are not restricted to them.
- SQL alerts can be attached to any Delta table reachable by a SQL query, including monitor-generated metrics tables.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** You are building a Databricks Asset Bundle to fully automate a drift-triggered retrain-validate-promote pipeline. Which set of bundle resources correctly implements this per recommended practice?

A) Only a monitoring resource, since retraining happens automatically without a Job definition.
B) A single notebook with no Job resource, run manually from the workspace UI each time drift is detected.
C) A Job resource with chained tasks for retrain, evaluate, and conditional promote, referencing model/monitor resources via bundle variables, deployed with `databricks bundle deploy`.
D) Only a cluster policy resource controlling which cluster sizes are allowed.

<details>
<summary>Answer</summary>

**Correct: C**

A DAB-defined Job with dependent tasks captures the full retrain -> validate -> promote flow as versioned, deployable infrastructure, which is the exam's expected automation pattern.

**Why the others are wrong:**
- A monitoring resource alone only computes metrics; it has no execution logic for retraining or promotion.
- A manually run notebook reintroduces the human step the automation goal is meant to remove.
- A cluster policy governs compute configuration only and performs no pipeline logic.

</details>

---

**Q8.** An exam scenario describes 'reacting automatically when a live production metric crosses a defined threshold and triggering remediation.' Which category of MLOps answer is being tested?

A) Monitoring plus automation (an alerting-triggered response), not pure lifecycle or testing.
B) Lifecycle governance only, referring to model stage transitions.
C) Testing only, referring to pre-deployment validation gates.
D) None of these — the correct answer always requires manual intervention.

<details>
<summary>Answer</summary>

**Correct: A**

Language about reacting to a live threshold crossing on a running system maps to monitoring + automation, distinct from lifecycle (stage/alias management) or testing (pre-deployment gates).

**Why the others are wrong:**
- Lifecycle language concerns alias/stage transitions, not reacting to a live metric threshold.
- Testing language concerns pre-promotion validation gates, not live production reaction.
- The scenario explicitly asks for automatic reaction, ruling out manual intervention as the intended answer.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

