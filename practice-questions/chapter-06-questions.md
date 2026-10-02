## Chapter 6: Model Lifecycle Management: MLflow Model Registry and Models in Unity Catalog — Practice Questions

**Q1.** A validated model must be referenced consistently by downstream inference jobs, and the team needs to swap which version is used in production instantly, without changing job code, while keeping a full audit trail in Unity Catalog. Which approach BEST meets these requirements?

A) Assign a 'Champion' alias to the validated version in the Unity Catalog registered model and reference models:/catalog.schema.model@Champion in job code
B) Use legacy MLflow Model Registry stages (Production) and reference models:/name/Production in job code
C) Hardcode the specific version number in job code and manually update it after each promotion
D) Reference the MLflow run ID directly using runs:/run_id/model in the job code

<details>
<summary>Answer</summary>

**Correct: A**

Aliases let job code stay static while pointing to whichever version is promoted, with full Unity Catalog audit and lineage.

**Why the others are wrong:**
- Legacy stages are deprecated and not supported for Unity Catalog three-level namespace models
- Manual version hardcoding breaks the 'swap without code change' requirement and lacks a governance trail
- Run IDs reference training runs, not registered/versioned models, and bypass the registry entirely

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** A model is registered and validated in a dev workspace and must be usable in a prod workspace with no artifact duplication. Both workspaces are attached to the same Unity Catalog metastore. What should the engineer do?

A) Export the model to DBFS in dev and manually copy the files to the prod workspace
B) Grant appropriate Unity Catalog permissions on the model in the target catalog/schema — cross-workspace access is automatic
C) Re-register the model separately in the prod workspace's local MLflow Model Registry
D) Set up a scheduled job to sync the workspace Model Registry between dev and prod

<details>
<summary>Answer</summary>

**Correct: B**

UC-registered models are metastore-level objects; any workspace attached to that metastore can access them once granted privileges like USE CATALOG, USE SCHEMA, and EXECUTE.

**Why the others are wrong:**
- Manual duplication is unnecessary and defeats the purpose of Unity Catalog
- Re-registering creates a disconnected duplicate and loses lineage back to the original training run
- Registry sync jobs are a legacy workaround made obsolete by Unity Catalog's native cross-workspace access

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A data scientist needs to load a Unity Catalog registered model version to run batch inference but does not need to modify the model or its aliases. Which privilege must be granted at minimum?

A) MODIFY
B) CREATE MODEL
C) OWNERSHIP of the model
D) EXECUTE

<details>
<summary>Answer</summary>

**Correct: D**

EXECUTE is the minimum privilege that allows loading/using a model version for inference without granting rights to change it.

**Why the others are wrong:**
- MODIFY-type change rights are not needed just to load a model for inference
- CREATE MODEL only allows creating new registered models in a schema, not using an existing one
- Ownership grants full control, which is far more access than required and violates least-privilege

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q4.** A new ML platform build in 2026 needs governed model versioning with cross-workspace access, fine-grained ACLs, and automatic lineage to training data and notebooks. Which should be used?

A) Legacy Workspace MLflow Model Registry with stage transitions
B) MLflow Tracking Server experiments only
C) A custom model metadata table stored in Delta
D) Models in Unity Catalog

<details>
<summary>Answer</summary>

**Correct: D**

Unity Catalog is Databricks' current recommended registry, providing account-level governance, cross-workspace sharing, and native lineage.

**Why the others are wrong:**
- The legacy registry lacks Unity Catalog ACLs and cross-workspace scope
- The tracking server logs runs and experiments but is not a governed model registry
- Reinventing registry functionality manually ignores native UC capabilities and lineage

</details>

---

**Q5.** Which URI correctly loads a model version using an alias from Unity Catalog?

A) models:/catalog.schema.model/1
B) models:/catalog.schema.model@Champion
C) models:/catalog.schema.model/Production
D) runs:/<run_id>/model

<details>
<summary>Answer</summary>

**Correct: B**

The @alias syntax is the Unity Catalog-supported way to reference a model version by alias.

**Why the others are wrong:**
- This references an explicit version number — valid syntax but not alias-based
- Stage-based syntax is not supported for Unity Catalog-registered models
- This references a training run's logged artifact, not a registry version

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** Before running mlflow.register_model() to target Unity Catalog instead of the legacy workspace registry, what must be configured?

A) mlflow.set_experiment('/Shared/uc-experiment')
B) mlflow.set_tracking_uri('databricks')
C) No configuration is needed — Unity Catalog is targeted automatically whenever a three-level model name is provided
D) mlflow.set_registry_uri('databricks-uc')

<details>
<summary>Answer</summary>

**Correct: D**

Setting the registry URI to 'databricks-uc' tells the MLflow client to target Unity Catalog rather than the legacy workspace registry.

**Why the others are wrong:**
- This only sets the experiment for tracking, unrelated to the registry target
- This sets the tracking URI for run logging, not the model registry destination
- Without setting registry_uri, register_model() defaults to the legacy workspace registry regardless of name format

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** What is the primary reason Unity Catalog aliases replace legacy Staging/Production/Archived stages for new model registrations?

A) Aliases are faster to query than stages
B) Aliases automatically deploy models to serving endpoints
C) Stages have been fully removed from MLflow and no longer exist in any form
D) Aliases are custom, reassignable, catalog-scoped pointers that work with Unity Catalog's cross-workspace governance and lineage, unlike workspace-scoped, fixed-lifecycle stages

<details>
<summary>Answer</summary>

**Correct: D**

The exam-tested distinction is governance scope and flexibility: aliases are named, mutable references governed by Unity Catalog ACLs across workspaces, while stages were rigid, workspace-only labels.

**Why the others are wrong:**
- Query performance is not the distinguishing factor tested
- Aliases do not trigger automatic deployment to serving endpoints
- Stages still exist for backward compatibility in the legacy Workspace Model Registry, but are deprecated for Unity Catalog

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q8.** A team has 5 versions of a registered model in Unity Catalog and wants to compare validation metrics before deciding which version becomes the new Challenger. What is the BEST approach?

A) Manually open each model version's source notebook to read printed metrics
B) Use mlflow.search_model_versions() combined with the linked run metrics, or compare runs in the MLflow UI
C) Delete all but the two best-looking versions to simplify comparison
D) Assign the Challenger alias to the newest version by default since it's most recent

<details>
<summary>Answer</summary>

**Correct: B**

MLflow tracking stores metrics per run, and search_model_versions/UI comparison allows systematic evaluation of all versions without guesswork.

**Why the others are wrong:**
- Manually reading notebooks is error-prone and not the recommended systematic practice
- Deleting versions is destructive and irreversibly loses lineage and history
- Newest is not necessarily best — the decision must be based on actual validation metrics

</details>

---

