## Architectural flow (what happens in order)
1. **Batch ingestion**: source data → MinIO `raw/` prefix (partitioned paths).
2. **Orchestration**: Airflow DAG triggers ingestion/ETL/dbt/tests and records results.
3. **Modeling**: dbt builds `stg_*` then `dim_*` / `fct_*` tables in the warehouse.
4. **Tests**: dbt tests validate constraints (not_null/unique/relationships).
5. **Run history**: write run-level metrics + status for observability and debugging.
6. **CI**: pull requests run basic checks (format/lint + dbt compile/tests as applicable).

```mermaid
flowchart LR
  subgraph SRC[Data Sources]
    CSV["CSV / Files"]
    API["External API"]
  end

  subgraph LZ[Landing Zone]
    S3["MinIO (S3-compatible)<br/>raw/bronze storage"]
  end

  subgraph ORCH[Orchestration]
    AF["Airflow DAGs<br/>schedule, retries, backfills"]
  end

  subgraph PROC[Processing + Modeling]
    ETL["Batch ETL<br/>(Python / Spark jobs)"]
    DBT["dbt<br/>(stg -> dim/fct + tests)"]
    DQ["Data Quality<br/>(dbt tests / Great Expectations optional)"]
  end

  subgraph DATA[Data Stores]
    WH["Postgres Warehouse<br/>raw_*, stg_*, dim_*, fct_*"]
    META["Metadata DB<br/>run_history, job_status, row_counts"]
  end

  subgraph OBS[Observability]
    LOGS["Structured Logs"]
    METRICS["Run Metrics<br/>duration, rows, failures"]
  end

  %% DATA PLANE (solid)
  CSV --> S3
  API --> S3
  S3 -->|read raw| ETL
  ETL -->|write raw/stg| WH
  DBT -->|build dim/fct| WH
  DQ -->|test results| META
  METRICS --> META

  %% CONTROL PLANE (dashed - Airflow triggers)
  AF -.->|trigger ingest| S3
  AF -.->|run ETL| ETL
  AF -.->|run dbt| DBT
  AF -.->|validate| DQ

  %% logs/metrics emissions (fine as solid)
  AF --> LOGS
  AF --> METRICS
  ETL --> LOGS
  DBT --> LOGS
```
