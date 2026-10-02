# Chapter 11 Lab: Deployment Strategies and Serving Patterns: Blue-Green, Canary, and Shadow

**Estimated time:** 25 minutes

## Exercise

Configure traffic-split percentages on a Databricks Model Serving endpoint to simulate a canary rollout between two model versions, query the endpoint, and review the endpoint's latency and error-rate metrics

## Key takeaway

Match the deployment strategy to risk tolerance (shadow for zero-risk evaluation, canary for gradual exposure, blue-green for instant cutover) and match inference mode to latency needs.
