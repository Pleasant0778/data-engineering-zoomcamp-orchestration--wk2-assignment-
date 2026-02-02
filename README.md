# NYC Taxi Data Pipeline with Kestra, Docker, and Google Cloud

This project implements a complete end-to-end batch data engineering pipeline using Kestra as the orchestrator, Docker Compose for local infrastructure, and Google Cloud Platform (GCS and BigQuery) as the data lake and warehouse. The pipeline ingests monthly NYC Yellow and Green Taxi trip data, stores raw files in Google Cloud Storage, and loads deduplicated, partitioned data into BigQuery tables on a schedule.

The design follows production-grade data engineering practices including idempotent loads, conditional branching, external tables, merge-based deduplication, centralized configuration, and workflow-driven infrastructure setup.

---

## Architecture Overview

NYC TLC CSV.gz files (GitHub Releases)  
→ Kestra Scheduled Workflow  
→ Google Cloud Storage (raw CSV)  
→ BigQuery External Tables  
→ BigQuery Partitioned Tables  
→ MERGE for deduplication  

-------

## Technologies Used

- Kestra (workflow orchestration)
- Docker & Docker Compose
- Google Cloud Storage
- BigQuery
- PostgreSQL (Kestra metadata & queue)
- Shell scripting
- SQL
- NYC TLC public dataset

---

## Google Cloud Configuration with Kestra KV

Kestra Open Source does not include a secrets manager. Instead, this project uses Kestra’s Key-Value (KV) store to manage Google Cloud configuration and authentication.

The full Google Cloud service account JSON key is stored as a STRING KV and injected into all GCP plugins automatically.

KV keys used:
- GCP_PROJECT_ID
- GCP_LOCATION
- GCP_BUCKET_NAME
- GCP_DATASET
- GCP_CRED (full service account JSON)

---

## GCP Setup Flow (KV Initialization)

```yaml
id: 01_gcp_setup
namespace: zoomcamp_learning

tasks:
  - id: gcp_project_id
    type: io.kestra.plugin.core.kv.Set
    key: GCP_PROJECT_ID
    kvType: STRING
    value: kestra-learning-486200  

  - id: gcp_location
    type: io.kestra.plugin.core.kv.Set
    key: GCP_LOCATION
    kvType: STRING
    value: europe-west2

  - id: gcp_bucket_name
    type: io.kestra.plugin.core.kv.Set
    key: GCP_BUCKET_NAME
    kvType: STRING
    value: zc-kestra-learning-486200-kestra  

  - id: gcp_dataset
    type: io.kestra.plugin.core.kv.Set
    key: GCP_DATASET
    kvType: STRING
    value: zoomcamp
```

---

## Scheduling with Timezone

```yaml
triggers:
  - id: green_schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 1 * *"
    timezone: America/New_York
    inputs:
      taxi: green
```

---

## Summary

This README consolidates all explanations, configuration, and code snippets discussed, into a single document for learning and reference.
