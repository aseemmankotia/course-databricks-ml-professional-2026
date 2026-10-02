# Chapter 4 Lab: Feature Engineering and Feature Store: Point-in-Time Correctness

**Estimated time:** 25 minutes

## Exercise

Create a time-series feature table with a timestamp key, build a training set using create_training_set with point-in-time FeatureLookups, and define an on-demand feature function computed at inference

## Key takeaway

Point-in-time correctness in feature lookups is the single most exam-tested concept in feature engineering — always join on the label timestamp, never the latest value.
