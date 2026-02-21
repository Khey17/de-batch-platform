# Batch Data Platform (in progress)

A production-shaped **batch data platform** that turns raw data into **analytics-ready datasets** using:
- landing zone (S3-style)
- orchestration (Airflow)
- transformations + modeling (dbt star schema)
- warehouse (Postgres/Redshift)
- data quality checks + run history

---

## Why this exists
- pipeline reliability (retries, idempotency, backfills)
- data modeling (dimensional/star schema)
- testing and validation
- observability (logs + run metrics)
- engineering hygiene (CI + clean docs)

---

## Definition of Done (MVP)
- [ ] `docker compose up` starts the stack locally
- [ ] Airflow DAG runs end-to-end
- [ ] Raw data lands in an S3-style layout (partitioned paths)
- [ ] Warehouse tables exist: `raw_*`, `stg_*`, `dim_*`, `fct_*`
- [ ] dbt builds + tests pass
- [ ] `run_history` captures run status + row counts + duration
- [ ] CI runs lint/tests/dbt checks on every push
- [ ] Demo assets exist: DAG screenshot + dbt lineage + query output

---

## Planned architecture (target)
See: `docs/architecture.md`

## Milestones
See: `docs/milestones.md`

## MVP Architecture

### What the MVP delivers (user-facing)
The MVP proves an end-to-end batch pipeline:
- Land raw files into an S3-style **landing zone** (MinIO).
- Orchestrate a repeatable workflow with **Airflow** (retries/backfills).
- Transform + model into **analytics-ready** tables with **dbt** (staging → star schema).
- Run **data quality checks** (dbt tests) on each run.
- Persist **run_history** (status, duration, row counts, failure reason).
