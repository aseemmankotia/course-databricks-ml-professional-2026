## Chapter 3: Advanced MLflow: Autologging, Nested Runs, Signatures, and Custom PyFunc — Practice Questions

**Q1.** A data scientist calls mlflow.autolog() then trains a custom PyTorch training loop (manual forward/backward/optimizer.step(), no Lightning Trainer or Estimator API). After training, the MLflow run shows no metrics or model artifact. What is the MOST likely cause?

A) mlflow.autolog() only works inside Databricks notebooks, not in scripts
B) mlflow.autolog() must be called after training completes, not before
C) PyTorch autologging patches high-level training APIs (e.g., Lightning Trainer); a manual loop bypasses those hooks so nothing is captured automatically
D) PyTorch is not a supported autologging flavor in MLflow

<details>
<summary>Answer</summary>

**Correct: C**

Autologging works by patching specific framework APIs (Trainer.fit, Estimator.fit, etc.). A raw manual loop never calls those patched methods, so no params/metrics/model are auto-captured — you must log manually inside the loop.

**Why the others are wrong:**
- Autologging works identically in notebooks and scripts
- autolog() must be called BEFORE the training code executes, not after — this is the opposite error
- PyTorch (via pytorch-lightning integration) is a supported flavor, just not for raw manual loops

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q2.** You are running a hyperparameter sweep with 20 trials and want each trial tracked as a child run under one parent run in the MLflow UI. Which code pattern is correct?

A) Call mlflow.start_run() 21 times in a row without ending any of them
B) Set `mlflow.autolog(nested=True)` and let autologging create the hierarchy automatically
C) Wrap the whole sweep in `with mlflow.start_run() as parent:` then call `with mlflow.start_run(nested=True):` inside the loop for each trial
D) Use 20 separate mlflow.start_run() calls each with the same run_id as the parent

<details>
<summary>Answer</summary>

**Correct: C**

Nested runs require an active parent run context; each child run inside the loop must explicitly pass nested=True or MLflow raises an exception for an already-active run.

**Why the others are wrong:**
- Starting multiple runs without ending them raises 'Run already active' errors
- autolog() has no nested parameter — nesting is controlled by start_run(), not autolog()
- Reusing the same run_id would overwrite the same run rather than create children

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q3.** A model requires a fitted StandardScaler to be applied to raw input before calling model.predict(). The team wants a single MLflow model that handles this internally when served, so downstream consumers can send raw, unscaled data. Which approach BEST satisfies this?

A) Log the scaler and the model as two separate mlflow.sklearn models and document that they must be called in order
B) Store the scaler parameters as run tags and instruct users to re-implement scaling manually
C) Retrain the model on unscaled data so scaling is no longer needed
D) Subclass mlflow.pyfunc.PythonModel, load the scaler in load_context(), apply it inside predict(), and log with mlflow.pyfunc.log_model(artifacts={'scaler': path})

<details>
<summary>Answer</summary>

**Correct: D**

Custom PyFunc lets you bundle preprocessing artifacts and logic into one deployable unit; load_context() loads the scaler from the artifacts dict and predict() applies it before inference — this is the standard exam pattern for packaging preprocessing.

**Why the others are wrong:**
- Two separate models pushes complexity to consumers and breaks single-artifact deployment
- Tags aren't executable and don't get invoked at inference time
- Retraining changes model behavior and doesn't address the packaging requirement

</details>

---

**Q4.** A team trains dozens of scikit-learn models daily and wants MLflow to automatically log parameters, metrics, and the model artifact without adding a single mlflow.log_* call. Which is the BEST fit?

A) mlflow.log_model() called manually after every training script
B) mlflow.sklearn.autolog() (or global mlflow.autolog()) called once before model.fit()
C) mlflow.pyfunc.PythonModel with a custom predict() method
D) Manually calling infer_signature() after each run

<details>
<summary>Answer</summary>

**Correct: B**

autolog() is purpose-built to eliminate manual logging calls for supported flavors like sklearn — it patches .fit() to automatically capture params, metrics, and the model.

**Why the others are wrong:**
- Manual log_model() defeats the 'no manual calls' requirement
- Custom PyFunc is for custom inference logic, not automatic logging
- infer_signature() only handles schema, not full autologging

</details>

---

**Q5.** You need the logged model's input/output schema to be automatically inferred AND validated against a sample of real data at logging time, so schema mismatches are caught immediately. Which logging approach accomplishes this?

A) Pass only a manually constructed ModelSignature object with no input_example
B) Skip signature entirely and rely on autolog() defaults
C) Log the schema as a JSON artifact separately from the model
D) Pass input_example to mlflow.<flavor>.log_model(); MLflow infers the signature from it and validates the example against the model

<details>
<summary>Answer</summary>

**Correct: D**

Providing input_example at log_model() time triggers automatic signature inference from real sample data and validates that the model can actually score that example, catching schema issues at logging time.

**Why the others are wrong:**
- A manually built signature with no example skips the validation step against real data
- autolog() defaults do NOT log input examples (log_input_examples defaults to False), so no validation occurs
- A separate JSON artifact isn't recognized by MLflow's schema enforcement at serving time

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q6.** By default, when you call plain mlflow.sklearn.autolog() with no arguments, which combination of behaviors is correct?

A) log_input_examples=True, log_model_signatures=True, log_models=True
B) log_input_examples=False, log_model_signatures=True, log_models=True
C) log_input_examples=False, log_model_signatures=False, log_models=False
D) log_input_examples=True, log_model_signatures=False, log_models=True

<details>
<summary>Answer</summary>

**Correct: B**

MLflow's autolog default flags are log_input_examples=False (to avoid storing potentially large/sensitive data by default) while log_model_signatures=True and log_models=True remain enabled by default.

**Why the others are wrong:**
- Input examples are NOT logged by default — a very commonly tested trap
- Model and signature logging are both True by default, not False
- Signatures ARE logged by default even though input examples are not

> ⚠️ Frequently missed on the exam — read the options carefully.

</details>

---

**Q7.** A student writes `mlflow.start_run(nested=True)` as the very first line of their script with no other run active. What happens?

A) MLflow automatically creates a parent run first, then nests the new run under it
B) MLflow raises an exception because nested=True requires an already-active parent run
C) The run is created as a top-level run and nested=True is silently ignored
D) MLflow creates the run inside the Default experiment regardless of nesting

<details>
<summary>Answer</summary>

**Correct: B**

nested=True is only valid when called inside the context of an already-active run; calling it with no active parent run raises an exception since there is nothing to nest under.

**Why the others are wrong:**
- MLflow does not auto-create a parent run — it errors out instead
- nested=True is not silently ignored; it enforces the requirement of a parent
- The experiment location is irrelevant to whether nesting requires an active parent

</details>

---

**Q8.** What is the fundamental difference between a model's logged signature and an input_example?

A) They are identical; input_example is just an alias for signature
B) Signature is only used for classification models, input_example only for regression
C) input_example enforces schema at inference time, while signature is purely documentation with no enforcement
D) The signature defines the expected input/output schema (names, types, shapes) used for validation at serving time; the input_example is a concrete sample of real data used to demonstrate and infer that schema

<details>
<summary>Answer</summary>

**Correct: D**

The signature is the formal schema contract enforced when serving (mlflow.pyfunc.load_model / model serving validates against it); input_example is a sample record stored alongside the model, often used to auto-generate that signature.

**Why the others are wrong:**
- They serve different purposes and are not aliases of each other
- Neither is restricted to a specific model task type
- It's the signature — not the input_example — that is enforced at serving time

</details>

---

