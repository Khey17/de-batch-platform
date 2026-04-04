# Batch Data Platform

A production-shaped **batch ELT platform** that ingests raw source data, models it into an analytics-ready Gold layer, and delivers trusted datasets for downstream analytics and science consumers.

Built on an AWS-native stack — S3, Lambda, Step Functions, Redshift Serverless, dbt, SNS, and CloudWatch. Infrastructure provisioned and torn down via Terraform.

---

## Stack

| Layer | Tool |
|---|---|
| Landing zone | S3 (partitioned bronze storage) |
| Orchestration | Step Functions |
| Ingestion + ETL | Lambda (Python) |
| Modeling | dbt |
| Warehouse | RDS Postgres (Redshift-compatible DDL) |
| Metadata | Glue Data Catalog |
| Data quality | dbt tests + custom Python |
| Alerting | SNS + CloudWatch |
| Infrastructure | Terraform |
| CI | GitHub Actions |

---

## Architecture

See: `docs/architecture.md`

---

## Data Model (Gold Layer)

Star schema designed for analytical query patterns:

```
fct_orders      — grain: one row per order
dim_customer    — customer attributes
dim_product     — product / listing attributes
```

Warehouse layer convention:
- `raw_*` — landed as-is from source
- `stg_*` — cleaned, typed, deduplicated
- `dim_*` — dimension tables
- `fct_*` — fact tables

---

## Definition of Done (MVP)

- [ ] Terraform provisions all AWS resources in one command
- [ ] Raw files land in S3 with partitioned paths (`raw/year=/month=/day=`)
- [ ] Step Functions state machine runs: ingest → ETL → dbt → quality checks → run_history
- [ ] Redshift DDL uses DISTKEY, SORTKEY, and compression encodings
- [ ] dbt builds `stg_*` → `dim_*` → `fct_*` with tests passing
- [ ] At least one data quality check fails a run and writes failure reason to `run_history`
- [ ] `run_history` captures: status, row counts, duration, failure reason
- [ ] SNS alert fires on pipeline failure
- [ ] Structured logs from all pipeline stages visible in CloudWatch
- [ ] CI runs dbt compile + lint on every push
- [ ] `terraform destroy` tears everything down cleanly
- [ ] Demo assets: pipeline run screenshot, dbt lineage graph, sample query output

---

## Milestones

See: `docs/milestones.md`
