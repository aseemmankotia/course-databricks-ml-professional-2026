## Chapter 6: Model Lifecycle Management: MLflow Model Registry and Models in Unity Catalog — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | Three-level namespace for registering a model in Unity Catalog? | catalog.schema.model_name (e.g., prod.ml_models.churn_predictor). Exam tip: replaces the single-level model name used in the legacy Workspace Model Registry. |
| 2 | MLflow registry URI setting required to register models into Unity Catalog? | mlflow.set_registry_uri('databricks-uc'). Without it, mlflow.register_model() targets the legacy workspace registry by default. |
| 3 | Command to set the Champion alias on version 3 of a UC-registered model? | client.set_registered_model_alias(name='catalog.schema.model', alias='Champion', version=3). Reassigning moves the pointer instantly — no new version created. |
| 4 | Correct UC URI syntax to load a model by alias vs by stage? | Alias (UC): models:/catalog.schema.model@Champion. Stage syntax models:/name/Production is legacy-only and NOT supported for Unity Catalog models. |
| 5 | Unity Catalog Model Registry vs legacy Workspace Model Registry — governance scope? | UC: metastore/account-level, cross-workspace, catalog/schema ACLs, native lineage. Legacy: workspace-scoped only, no cross-workspace sharing. |
| 6 | Aliases vs Stages — key structural difference? | Stages (Staging/Production/Archived) are fixed, single-assignment lifecycle labels. Aliases are custom, unlimited named pointers (Champion, Challenger, Baseline) reassignable to any version at any time. |
| 7 | Why are legacy MLflow Model Registry stages deprecated for new workflows? | Stages are workspace-scoped, lack fine-grained Unity Catalog ACLs/lineage, and enforce rigid lifecycle naming. Databricks recommends Unity Catalog + aliases for all new registrations. |
| 8 | How does a model registered in Unity Catalog become available in a different workspace? | Any workspace attached to the same Unity Catalog metastore has access automatically, subject to grants — no copying or re-registration needed. |
| 9 | Champion/Challenger pattern — typical governance workflow? | New candidate version registered → evaluated → 'Challenger' alias assigned → after validation passes, 'Champion' alias is reassigned to that version → old Champion version stays in registry with full lineage. |
| 10 | UC privilege needed to load/invoke a registered model version for inference? | EXECUTE privilege on the model. Separate from privileges needed to change aliases/tags, which require model ownership or broader grants (e.g., ALL PRIVILEGES). |
| 11 | How many aliases can point to the same model version simultaneously in Unity Catalog? | Multiple aliases (e.g., Champion AND Challenger) can reference the same version, but each individual alias points to only ONE version at a time. |
| 12 | API to compare model versions before promotion? | mlflow.search_model_versions("name='catalog.schema.model'") lists versions/metadata; use the MLflow UI 'Compare Runs' view to diff metrics/params of the linked training runs. |
