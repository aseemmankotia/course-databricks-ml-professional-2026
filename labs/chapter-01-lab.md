# Chapter 1 Lab: Single-Node vs Distributed Training: Choosing the Right Approach

**Estimated time:** 20 minutes

## Exercise

Build a Spark ML Pipeline (VectorAssembler + RandomForestClassifier) on a Databricks cluster and compare training time/behavior against single-node scikit-learn on the driver

## Key takeaway

Choose distributed Spark ML only when data or compute genuinely exceeds single-node capacity — otherwise single-node training is simpler and often faster.
