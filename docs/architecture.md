```mermaid
flowchart LR
  %% -------------------
  %% SOURCES
  %% -------------------
  subgraph SRC[Data Sources]
    CSV[CSV / Files]
    API[External API]
  end

  %% -------------------
  %% LANDING ZONE
  %% -------------------
  subgraph LZ[Landing Zone]
    S3[(MinIO - S3 compatible<br/>raw/bronze storage)]
  end

  %% -------------------
  %% ORCHESTRATION
  %% -------------------
  subgraph ORCH[Orchestration]
    AF[Airflow DAGs<br/>(schedule, retries, backfills)]
  end

  %% -------------------
  %% PROCESSING + MODELING
  %% -------------------
  subgraph PROC[Processing + Modeling]
    ETL[Batch ETL<br/>(Python / Spark jobs)]
    DBT[dbt<br/>(stg -> dim/fct + tests)]
    DQ[Data Quality<br/>(dbt tests / Great Expectations optional)]
  end

  %% -------------------
  %% WAREHOUSE + METADATA
  %% -------------------
  subgraph DATA[Data Stores]
    WH[(Postgres Warehouse<br/>raw_*, stg_*, dim_*, fct_*)]
    META[(Postgres Metadata<br/>run_history, job_status, row_counts)]
  end

  %% -------------------
  %% OBSERVABILITY
  %% -------------------
  subgraph OBS[Observability]
    LOGS[Structured Logs]
    METRICS[Run Metrics<br/>(duration, rows, failures)]
  end

  %% FLOW
  CSV --> S3
  API --> S3

  AF -->|1) ingest/land| S3
  AF -->|2) run ETL| ETL
  ETL -->|write raw/stg| WH

  AF -->|3) run dbt| DBT
  DBT -->|build dim/fct| WH

  AF -->|4) validate| DQ
  DQ -->|test results| META

  AF --> LOGS
  AF --> METRICS
  ETL --> LOGS
  DBT --> LOGS
  METRICS --> META
```