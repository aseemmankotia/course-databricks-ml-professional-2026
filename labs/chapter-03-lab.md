# Chapter 3 Lab: Advanced MLflow: Autologging, Nested Runs, Signatures, and Custom PyFunc

**Estimated time:** 25 minutes

## Exercise

Wrap a preprocessing pipeline plus a trained model in a custom mlflow.pyfunc.PythonModel, log it with a signature and input example, then reload and score it to confirm parity with the original pipeline

## Key takeaway

Custom PyFunc models with explicit signatures and nested-run tracking are the professional-level MLflow patterns the exam expects.
