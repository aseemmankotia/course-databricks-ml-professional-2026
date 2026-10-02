## Chapter 1: Single-Node vs Distributed Training: Choosing the Right Approach — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | Spark ML Estimator vs Transformer — which method defines each? | Estimator implements .fit() → returns a Model (which is itself a Transformer). Transformer implements .transform(). Exam tip: Spark ML has no fit_transform() like scikit-learn. |
| 2 | Default value of spark.sql.autoBroadcastJoinThreshold? | 10 MB. Tables smaller than this are auto-broadcast to all executors, avoiding a shuffle. Use the broadcast() hint to force it for larger tables. |
| 3 | Decision rule: single-node library vs Spark ML? | If the full training dataset fits in the memory of one node → use single-node library (scikit-learn/XGBoost). If it does not fit → use a Spark ML data-parallel algorithm. Decision is based on DATA size, not cluster size. |
| 4 | How to parallelize hyperparameter search for a single-node model (e.g. XGBoost) across a cluster? | Hyperopt + SparkTrials — distributes independent single-node training trials across worker nodes. Use regular Trials (not SparkTrials) for models that are already Spark-distributed. |
| 5 | Pandas API on Spark (pyspark.pandas) — main use case? | Scaling pandas-style feature engineering/data prep across a Spark cluster without rewriting code as PySpark DataFrame API. Convert the small final result to pandas before using scikit-learn. |
| 6 | Data-parallel vs Model-parallel training? | Data-parallel: dataset split across nodes, each node has full model copy (e.g. Spark ML). Model-parallel: model itself split across nodes because it's too large for one machine. |
| 7 | Best cluster type for single-node ML training (sklearn/XGBoost)? | A single-node cluster (driver only, no workers) sized with enough RAM/cores for the dataset. Multi-worker clusters leave workers idle since single-node libraries never distribute across Spark. |
| 8 | Spark ML Param — what is it? | A named, documented hyperparameter attached to an Estimator/Transformer (e.g. maxIter, regParam), settable via setParams() and usable in ParamGridBuilder for tuning/CV. |
| 9 | Syntax to force a broadcast join in PySpark? | from pyspark.sql.functions import broadcast; df.join(broadcast(small_df), 'key') — forces the small DataFrame to be copied to every executor. |
| 10 | GBTClassifier (Spark MLlib) vs GradientBoostingClassifier (scikit-learn) — when to use which? | GBTClassifier: data-parallel, use when data doesn't fit on a single node. GradientBoostingClassifier: single-node, use when data fits in memory. Choice is driven by data size. |
| 11 | Spark ML Pipeline execution order? | Pipeline(stages=[...]).fit(df) runs each stage in order: Estimators call .fit() then .transform(); Transformers call only .transform(). Result is a PipelineModel. |
| 12 | Common exam trap: using SparkTrials with a Spark ML (already-distributed) model? | WRONG. SparkTrials is only for parallelizing single-node model trials across workers. For Spark ML distributed models, use Hyperopt's plain Trials class since training is already distributed. |
