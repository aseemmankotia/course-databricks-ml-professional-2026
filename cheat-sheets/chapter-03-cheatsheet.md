## Chapter 3 Cheat Sheet — Advanced MLflow

### Autologging
| Setting | Default | Notes |
|---|---|---|
| log_models | True | Auto-logs model artifact |
| log_model_signatures | True | Auto-infers schema |
| log_input_examples | **False** | Must opt in — exam trap |
| Call timing | **Before** .fit() | Patches training methods |

**Key rule:** Autologging only works when you call a *patched* API (e.g., `.fit()`, Lightning `Trainer.fit()`). Manual/custom training loops log nothing automatically — use `mlflow.log_metric()` / `log_param()` explicitly.

```python
import mlflow
mlflow.autolog()  # or mlflow.sklearn.autolog(log_input_examples=True)
model.fit(X_train, y_train)
```

### Nested Runs
```python
with mlflow.start_run(run_name="sweep") as parent:
    for params in param_grid:
        with mlflow.start_run(nested=True):
            mlflow.log_params(params)
            mlflow.log_metric("accuracy", acc)
```
- `nested=True` **requires an active parent run** — else raises exception.
- Pattern: parent = sweep summary, children = individual trials.

### Model Signature & Input Example
| Concept | Purpose | Enforced at serving? |
|---|---|---|
| Signature | Schema (names/types/shapes) | **Yes** |
| input_example | Sample record | No (used to derive/validate signature) |

```python
from mlflow.models.signature import infer_signature
sig = infer_signature(X_train, model.predict(X_train))
mlflow.sklearn.log_model(model, "model", signature=sig, input_example=X_train.iloc[:5])
```
Providing `input_example` at log time triggers automatic signature inference AND validates the model can actually score it — catches mismatches immediately.

### Custom PyFunc
```python
class MyModel(mlflow.pyfunc.PythonModel):
    def load_context(self, context):
        self.scaler = joblib.load(context.artifacts["scaler"])
        self.model = joblib.load(context.artifacts["model"])
    def predict(self, context, model_input):
        scaled = self.scaler.transform(model_input)
        return self.model.predict(scaled)

mlflow.pyfunc.log_model(
    artifact_path="model",
    python_model=MyModel(),
    artifacts={"scaler": "scaler.pkl", "model": "model.pkl"},
    input_example=sample_df
)
```
- `predict(self, context, model_input)` is **required**.
- `load_context(self, context)` is optional — loads bundled artifacts before inference.
- Use custom PyFunc to package preprocessing + model as ONE deployable unit.

### Common Exam Traps
1. Thinking autolog captures manual training loops — it doesn't.
2. Forgetting `nested=True` needs an active parent run first.
3. Assuming input_example is logged by default — it is NOT.
4. Confusing signature (enforced) with input_example (sample only).
5. Forgetting predict() signature must be `(self, context, model_input)`.
