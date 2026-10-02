## Chapter 7: CI/CD for ML: Databricks Asset Bundles, Jobs, and Automated Retraining — Flashcards

| # | Front (Question) | Back (Answer) |
|---|-----------------|---------------|
| 1 | Which file defines a Databricks Asset Bundle's resources and targets? | databricks.yml — contains bundle, resources, targets, variables, include keys. Exam tip: resources = what to deploy, targets = where/how per environment. |
| 2 | Command to deploy a Databricks Asset Bundle to the 'prod' target? | databricks bundle deploy -t prod. Exam tip: always specify -t explicitly in CI/CD scripts to avoid deploying to the wrong environment. |
| 3 | Command to run a specific job defined in a bundle after deployment? | databricks bundle run -t <target> <job-resource-key>. Exam tip: deploy first, then run — two separate steps. |
| 4 | What does mode: development do in a databricks.yml target? | Prefixes resource names with the deploying user's identity and pauses schedules/triggers by default, isolating dev deployments from shared/production resources. |
| 5 | How do you parameterize a bundle so cluster size differs between dev and prod? | Define a variables: block at bundle root with defaults, override under targets.<name>.variables, and reference it in resources with ${var.<name>}. |
| 6 | File arrival vs Continuous vs Scheduled trigger — when to use each for retraining? | File arrival = fires when new data lands in a storage path (event-driven). Continuous = reruns immediately after previous run finishes. Scheduled = fixed cron interval. Exam tip: 'retrain when new data arrives' → file arrival, not continuous. |
| 7 | How does a multi-task Job stop downstream deploy tasks when evaluation fails a threshold? | Evaluate task raises an exception/fails when the metric is below threshold; downstream tasks using depends_on with the default run_if: ALL_SUCCESS are automatically skipped. |
| 8 | What MLflow Model Registry feature enables automated champion/challenger promotion? | Registered model aliases (@champion, @challenger). An evaluation task compares run metrics via the MLflow API and reassigns the @champion alias programmatically if the challenger passes a defined threshold. |
| 9 | Manually configured Jobs (UI) vs Databricks Asset Bundles — key CI/CD difference? | Bundles are declarative, git-versioned IaC deployable identically across dev/staging/prod via CLI; UI-created jobs are imperative and hard to reproduce consistently across environments. |
| 10 | Which databricks.yml top-level key holds job/pipeline/model definitions to deploy? | resources: — contains resources.jobs, resources.pipelines, resources.experiments, resources.models, resources.model_serving_endpoints. |
| 11 | Command to check a bundle's config is valid before deploying? | databricks bundle validate -t <target>. Exam tip: run in CI before the deploy step to catch config errors early. |
| 12 | Typical CI/CD git flow for ML bundles across environments? | PR merge to main → validate + deploy staging target + run integration test job. Git tag/release → deploy prod target (run_as service principal recommended). Exam tip: environments map to bundle targets, not separate repos. |
