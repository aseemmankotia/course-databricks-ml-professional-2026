## Chapter 7: CI/CD for ML: Databricks Asset Bundles, Jobs, and Automated Retraining — Practice Questions

**Q1.** A team wants to retrain a model whenever new labeled data lands in a Delta table's underlying cloud storage location, without polling on a fixed interval. Which Databricks Job trigger type should they configure?

A) Continuous trigger
B) Scheduled trigger with a cron expression running every 5 minutes
C) Manual trigger invoked only via REST API
D) File arrival trigger pointing to the storage location

<details>
<summary>Answer</summary>

**Correct: D**

File arrival triggers monitor a specific cloud storage path and fire only when new files land, giving event-driven retraining without polling overhead.

**Why the others are wrong:**
- Continuous trigger reruns the job immediately after the previous run finishes, regardless of data arrival — not tied to new data.
- A 5-minute cron introduces latency and unnecessary compute cost compared to an event-driven trigger.
- A manual-only trigger is not automated and defeats the goal of automated retraining.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** Which capability is the primary value-add of using Databricks Asset Bundles for CI/CD, compared to manually creating Jobs through the Databricks UI?

A) Ability to schedule jobs using cron expressions
B) Ability to run multi-task jobs with dependencies
C) Ability to log metrics to MLflow during job runs
D) Declarative, version-controlled infrastructure-as-code definitions deployable consistently across environments via the CLI

<details>
<summary>Answer</summary>

**Correct: D**

Bundles exist to provide git-versioned, declarative IaC that can be deployed identically to dev, staging, and prod using databricks bundle deploy.

**Why the others are wrong:**
- Cron scheduling is a Jobs feature available whether or not bundles are used.
- Multi-task dependencies are a Jobs feature independent of bundles.
- MLflow logging happens inside notebooks/scripts regardless of how the job was created.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** In a databricks.yml file, which top-level key defines environment-specific overrides such as workspace URL, deployment mode, and variable values for dev, staging, and prod?

A) resources
B) include
C) targets
D) bundle

<details>
<summary>Answer</summary>

**Correct: C**

The targets block defines per-environment configuration such as mode, workspace, run_as, and variable overrides.

**Why the others are wrong:**
- resources defines the jobs/pipelines/models themselves, not environment overrides.
- include is used to split the bundle configuration across multiple YAML files.
- bundle simply names/identifies the bundle project.

</details>

---

**Q4.** A retraining pipeline registers a new model version with alias @challenger in the MLflow Model Registry. The pipeline must automatically promote it to @champion only if a target metric improves by at least 2% on a held-out validation set. What is the BEST implementation approach?

A) Add an evaluation task that pulls champion and challenger metrics via the MLflow API, compares them programmatically, and conditionally calls the Model Registry API to reassign the @champion alias if the threshold is met
B) Manually review the run in the MLflow UI each cycle and reassign the alias when appropriate
C) Configure the job to always reassign @champion to the most recently registered version
D) Increase the scheduled trigger interval so promotion reviews happen less frequently

<details>
<summary>Answer</summary>

**Correct: A**

Automated promotion requires programmatic metric comparison against a defined threshold followed by a conditional Registry API call to update the alias — this matches the documented automated promotion pattern.

**Why the others are wrong:**
- Manual review is not automation and fails the requirement.
- Always promoting the latest version skips validation entirely and risks deploying a worse model.
- Changing the trigger interval does not address the comparison/promotion logic at all.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q5.** A multi-task Job has tasks train_task -> evaluate_task -> register_task -> deploy_task, each depending on the prior task's success. The evaluate_task determines the model does not meet the accuracy threshold. What configuration correctly prevents register_task and deploy_task from running while accurately reflecting the failure in the job run status?

A) Remove all task dependencies so tasks run in parallel
B) Keep the default depends_on with run_if: ALL_SUCCESS, and have evaluate_task raise an exception when the threshold is not met so downstream tasks are automatically skipped and the run shows failed
C) Add a fixed sleep/delay inside evaluate_task
D) Configure deploy_task to run unconditionally regardless of upstream task state

<details>
<summary>Answer</summary>

**Correct: B**

The default ALL_SUCCESS run_if condition combined with depends_on means an exception in evaluate_task causes downstream tasks to be skipped and the overall run to fail, correctly gating deployment on evaluation success.

**Why the others are wrong:**
- Removing dependencies breaks orchestration order and could deploy an unevaluated model.
- A sleep/delay has no effect on conditional execution logic.
- An unconditional deploy_task defeats the purpose of gating deployment on evaluation results.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** You need the training task's cluster size and number of epochs to differ between the dev and prod targets in a bundle. Which databricks.yml feature accomplishes this cleanly?

A) Duplicate the job definition into separate resources blocks for dev and prod
B) Maintain separate YAML files per environment and manually select the correct one before each deploy
C) Rely solely on shell environment variables with no reference in databricks.yml
D) Define bundle-level variables with default values, override them under each target's variables mapping, and reference them in resources using ${var.<name>}

<details>
<summary>Answer</summary>

**Correct: D**

This is the documented pattern: declare variables at the bundle root with defaults, override per target, and interpolate with ${var.name} in resource definitions.

**Why the others are wrong:**
- Duplicating job definitions violates DRY IaC principles and is hard to maintain.
- Manually selecting files before deploy is error-prone and not part of the bundle abstraction.
- Bundles require explicit variable references in the YAML; plain shell env vars alone aren't consumed automatically.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** What does deploying a bundle target configured with mode: development do differently than mode: production?

A) It disables MLflow logging entirely for the deployment
B) It prefixes deployed resource names with the deploying user's identity and pauses schedules/triggers by default, isolating the deployment from shared resources
C) It automatically deletes all resources after each job run
D) It forces every job to run exclusively on serverless compute

<details>
<summary>Answer</summary>

**Correct: B**

Development mode namespaces resources per user and pauses schedules/triggers by default to prevent collisions with shared or production resources.

**Why the others are wrong:**
- Development mode has no effect on MLflow logging behavior.
- Bundles do not auto-delete resources after runs; that requires an explicit bundle destroy.
- Compute type is defined per resource/cluster config, not dictated by target mode.

</details>

---

**Q8.** A CI/CD pipeline runs on every pull request merge to main: it validates the bundle, deploys to the staging target, and runs an integration test job. A separate release pipeline triggered by a git tag deploys to prod. Which command sequence correctly performs the staging deployment step in the PR pipeline?

A) databricks bundle deploy -t staging, followed by databricks bundle run -t staging integration_test_job
B) databricks jobs create --json-file staging_job.json
C) databricks bundle deploy with no target flag, assuming it defaults to prod
D) databricks bundle validate -t staging only, without a deploy step

<details>
<summary>Answer</summary>

**Correct: A**

The correct CLI workflow is to deploy the bundle to the named target and then run the specific job resource key by name.

**Why the others are wrong:**
- Creating jobs directly via the Jobs API bypasses the bundle abstraction and is not IaC-managed.
- Omitting -t risks deploying to an unintended default target rather than explicit staging.
- Validation alone does not deploy resources, so the integration test job cannot run afterward.

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

