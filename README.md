 PySpark and Databricks-oriented reference implementation for a regulated payments platform. It demonstrates how raw payment and counterparty data can move through bronze, silver, and gold layers while staying aligned with quality controls, auditability, privacy requirements, and regulatory evidence generation.

The repository is intentionally broad. It combines ingestion, transformation, compliance logic, observability, orchestration, infrastructure, simulations, contracts, and tests so the whole lifecycle can be exercised in one place.

## What This Project Covers

FRCE is organized around a few connected concerns:

- Payments ingestion from files and streaming sources.
- Exchange-rate ingestion from the ECB.
- Counterparty master data with SCD2 handling.
- Data quality validation and quarantine handling.
- AML alert generation and rule evaluation.
- GDPR erasure processing and audit reporting.
- DORA incident classification and evidence capture.
- EU AI Act style model transparency, inference logging, and model cards.
- Databricks bundle, catalog, storage, access, and masking infrastructure.

## Reference Architecture

The codebase follows a layered lakehouse flow:

1. Raw data lands in bronze tables and landing paths.
2. Standardization and business rules produce silver tables.
3. Curated marts and facts in gold support reporting and evidence.
4. Compliance pipelines write audit records and structured evidence.
5. Observability modules track lineage, health, SLAs, and DQ outcomes.

That layering is reflected in the Python package layout:

- `src/frce/ingestion` for source-specific loaders and streaming entry points.
- `src/frce/transformations` for bronze-to-silver normalization and masking helpers.
- `src/frce/quality` for expectations, rules, validation, and quarantine.
- `src/frce/gold` for marts and analytical outputs.
- `src/frce/compliance` for AML, GDPR, DORA, and EU AI Act logic.
- `src/frce/observability` for lineage, metrics, SLAs, and evidence reporting.
- `src/frce/audit` for pipeline run logging and evidence persistence.
- `src/frce/contracts` for schema and contract validation.
- `src/frce/intelligence` for feature engineering, anomaly scoring, and model governance.
- `src/frce/orchestration` for task registration and dependency execution.

## Main Workflows

### Batch Processing

The batch pipeline is the primary orchestration path and is exposed through `frce-batch`. It typically coordinates:

- raw ingestion
- normalization
- data quality checks
- compliance enrichment
- gold table materialization
- audit and lineage writes

### Streaming Payments

The streaming path is exposed through `frce-streaming` and centers on payments arriving through Kafka and landing into bronze storage before being promoted downstream.

### GDPR Erasure

The erasure flow is exposed through `frce-gdpr-erasure` and supports privacy requests, erasure audit tracking, and evidence generation for completed actions.

### DORA Incident Classification

The incident classifier is exposed through `frce-incident-classifier` and classifies operational incidents for DORA-style reporting and register maintenance.

## Repository Layout

- `apps/` contains runnable application entry points.
- `src/frce/` contains the reusable package code.
- `tests/unit/` contains unit coverage for transformations, compliance rules, utilities, and orchestration.
- `tests/integration/` contains end-to-end validation for higher-level flows.
- `config/` contains environment-specific configuration examples.
- `infra/` contains Terraform, SQL bootstrap assets, and Databricks bundle resources.
- `policies/` contains masking, access, retention, and OPA policy definitions.
- `contracts/` contains YAML contracts for governed inputs and outputs.
- `data_products/` contains ownership, SLA, quality, lineage, and compliance metadata.
- `evidence/` contains report templates and SQL queries used to assemble regulatory evidence.
- `simulations/` contains scenario inputs and synthetic workload generators.
- `benchmarks/` contains performance experiments for selected workloads.
- `docs/` contains runbooks and compliance mapping notes.
- `security/` contains threat modeling, secret-handling, and data classification guidance.

## Key Components

### Ingestion

The ingestion layer includes:

- `src/frce/ingestion/payments_stream_bronze.py` for streaming payment ingestion.
- `src/frce/ingestion/ecb_rates_ingestion.py` and `src/frce/ingestion/ecb_rates_bronze.py` for FX reference data.
- `src/frce/ingestion/counterparty_ingestion.py` and `src/frce/ingestion/counterparty_bronze.py` for counterparty source data.
- Kafka and ECB client wrappers under `src/frce/clients/`.

### Transformations

Normalization and masking live in:

- `src/frce/transformations/silver_payments.py`
- `src/frce/transformations/silver_fx.py`
- `src/frce/transformations/silver_counterparty.py`
- `src/frce/transformations/silver_counterparty_scd2.py`
- `src/frce/transformations/pii_tagger.py`
- `src/frce/transformations/pii_tagging.py`

### Gold Layer

Curated marts and facts live under `src/frce/gold/` and include:

- payment facts
- counterparty dimensions
- AML alerts
- DORA incident marts
- GDPR request marts
- model registry marts

### Compliance

Compliance-focused logic includes:

- `src/frce/compliance/aml/` for alerting and rule evaluation.
- `src/frce/compliance/gdpr/` for erasure pipelines and PII controls.
- `src/frce/compliance/dora/` for incident registration and classification.
- `src/frce/compliance/eu_ai_act/` for model cards, inference logging, and transparency tracking.
- `src/frce/compliance/model_card_writer.py` and related governance helpers.

### Observability and Audit

Operational visibility is supported through:

- lineage capture
- health checks
- SLA monitoring
- metrics reporting
- evidence report generation
- pipeline audit logging

### Contracts and Quality

The repo includes contract and rule artifacts for governed data products:

- `src/frce/contracts/` and the top-level `contracts/` directory.
- `src/frce/quality/` for DQ rules and validation.
- `data_products/*/contract.yml`, `quality.yml`, `ownership.yml`, `lineage.yml`, and related metadata.

## Configuration

Runtime settings are defined in `src/frce/config/settings.py` and read from environment variables prefixed with `FRCE_`.

Common settings include:

- `FRCE_ENVIRONMENT`
- `FRCE_CATALOG`
- `FRCE_STORAGE_ACCOUNT`
- `FRCE_KAFKA_BOOTSTRAP_SERVERS`
- `FRCE_KAFKA_PAYMENTS_TOPIC`
- `FRCE_ECB_BASE_URL`
- `FRCE_DATABRICKS_HOST`
- `FRCE_DATABRICKS_TOKEN`
- `FRCE_ANOMALY_MODEL_URI`

Environment examples live in:

- `config/local.yml`
- `config/dev.yml`
- `config/staging.yml`
- `config/prod.yml`

The table naming and storage path conventions are centralized in `FrceConfig`, which builds bronze, silver, gold, audit, and compliance locations from the selected catalog and storage account.

## Databricks And Infra

This repository includes the infrastructure scaffolding needed to deploy the project into a Databricks-backed environment:

- `databricks.yml`
- `infra/bundle/`
- `infra/*.tf`
- `infra/sql/*.sql`
- `infra/environments/*.tfvars`

These assets define catalogs, storage, grants, masking functions, ABAC policies, and workspace/job resources for the batch, streaming, GDPR, and ML workflows.

## Evidence And Compliance Artifacts

Regulatory evidence is first-class in this project. Useful assets include:

- `evidence/templates/` for report templates.
- `evidence/queries/` for SQL extracts feeding those reports.
- `docs/dora_compliance_map.md` for reporting alignment.
- `docs/eu_ai_act_evidence.md` for model governance evidence.
- `docs/gdpr_erasure_runbook.md` for operational privacy handling.
- `security/threat_model.md` and `security/data_classification.yml` for governance context.

## Simulations And Benchmarks

The repository includes synthetic data generation and scenario-driven exercises:

- `simulations/payments/`
- `simulations/counterparties/`
- `simulations/gdpr/`
- `simulations/incidents/`
- `simulations/chaos/`

Benchmarks under `benchmarks/` help exercise specific workloads such as erasure, AML scoring, and DORA incident queries.

## Developer Setup

The project targets Python 3.11.

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -e ".[dev]"
```

Run the test suite with:

```powershell
pytest -q
```

If you want to check formatting and linting locally, use the tools configured in `pyproject.toml`.

## Command-Line Entry Points

After installing the package, the following console scripts are available:

```powershell
frce-batch
frce-streaming
frce-gdpr-erasure
frce-incident-classifier
```

These map to the orchestration and application modules in `src/frce/` and `apps/`.

## How To Read The Code

If you are new to the repo, a good reading order is:

1. `src/frce/config/settings.py`
2. `src/frce/orchestration/pipeline_runner.py`
3. `src/frce/ingestion/`
4. `src/frce/transformations/`
5. `src/frce/quality/`
6. `src/frce/gold/`
7. `src/frce/compliance/`
8. `src/frce/observability/`

That path shows how configuration flows into ingestion, how tables are prepared, and how compliance and reporting artifacts are produced.

## Notes

- The repo is designed as a reference implementation, so some paths are illustrative and expect external infrastructure.
- Databricks catalog, storage, secrets, and workspace access must be configured before deployment.
- The project intentionally keeps governance artifacts close to the code so evidence and implementation can evolve together.

