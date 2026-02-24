---
layout: default
title: "DataGeni — Solution Architecture & Implementation"
description: "Detailed architecture, component design, API contracts, data model, and implementation guide for DataGeni."
---

[← Back to Platform Architecture](../architecture.md) · [Requirements Spec](../solutions/datageni.md) · [Agent Design](../agents/test-data-generator.md)
{: .fs-3 }

# DataGeni — Solution Architecture & Implementation

---

## 1. Architecture Overview

<img src="../../assets/images/arch-datageni.svg" alt="DataGeni Architecture" width="100%" style="border-radius:6px; margin-bottom:1rem;" />

```mermaid
flowchart LR
    CG[CaseGeni<br/>Data Needs] --> UI

    subgraph Users
        AE([Automation Engineer])
        PE([Performance Engineer])
    end

    subgraph DataGeni
        UI[DataGeni UI<br/>Schema · Config · Export]
        API[DataGeni API]
    end

    subgraph Agent
        TDG[Test Data Generator<br/>Synthetic · Edge · FK · PII-safe]
    end

    subgraph Tools
        FK[Faker / Mimesis]
        PII[Presidio PII Scan]
    end

    subgraph Models["Model Providers"]
        CLA[Claude]
        GEM[Gemini]
    end

    DB[(Blob Storage<br/>Datasets · Seeds)]

    Users --> UI --> API --> TDG
    TDG --> CLA & GEM
    TDG --> FK & PII
    TDG -->|datasets| API
    API --> DB

    API -.->|fixtures| APP[Auto-PlayPilot]
    API -.->|load data| PX[Perf-Xi]
```

### Key Flows

- Schemas imported from DDL, JSON Schema, or DB introspection
- The **Test Data Generator** uses Faker/Mimesis for realistic values + **Presidio** for PII compliance
- Referential integrity (FK) enforced across multi-table datasets
- Exports: CSV, JSON, SQL INSERT, Parquet — up to 1 M rows
- Datasets feed **Auto-PlayPilot** (fixtures) and **Perf-Xi** (load data)

---

## 2. Component Breakdown

```mermaid
flowchart TB
    subgraph FrontEnd["DataGeni UI (React / Next.js)"]
        SCHEMA[Schema Designer<br/>Tables · Columns · FKs]
        CONFIG[Config Panel<br/>Row count · Locale · Edge %]
        PREVIEW[Data Preview<br/>Sample rows · Validation]
        EXPORT[Export Manager<br/>CSV · JSON · SQL · Parquet]
    end

    subgraph APILayer["DataGeni API (FastAPI)"]
        INGEST[Schema Ingestion<br/>DDL · JSON Schema · DB introspect]
        GENSVC[Generation Service<br/>Agent orchestration]
        FKSVC[FK Resolver<br/>Cross-table integrity]
        PIISVC[PII Scanner<br/>Presidio compliance]
        EXPSVC[Export Service<br/>Format conversion · streaming]
    end

    subgraph AgentLayer["Test Data Generator Agent"]
        SYNTH[Synthetic Value Generator]
        EDGEGEN[Edge-Case Injector]
        FKENG[FK Constraint Engine]
        PIICHK[PII Compliance Checker]
    end

    subgraph DataLayer["Data Stores"]
        BLOB[(Blob Storage<br/>Generated datasets)]
        PG[(PostgreSQL<br/>Schemas · Jobs · Configs)]
        REDIS[(Redis<br/>Job queue · Preview cache)]
    end

    FrontEnd --> APILayer
    APILayer --> AgentLayer
    APILayer --> DataLayer
```

| Component | Technology | Responsibility |
|---|---|---|
| **DataGeni UI** | React 18 / Next.js 15, Tailwind CSS | Schema design, configuration, preview, export |
| **DataGeni API** | Python 3.12, FastAPI, Uvicorn | Schema ingestion, generation orchestration, FK resolution, export |
| **Test Data Generator** | Python 3.12, LangChain, Faker, Mimesis | Synthetic data, edge-case injection, PII compliance |
| **PostgreSQL** | PostgreSQL 16 | Schema definitions, job tracking, config persistence |
| **Blob Storage** | Azure Blob / S3 | Large dataset storage (CSV, Parquet) |
| **Redis** | Redis 7 | Async job queue, preview caching |

---

## 3. Tech Stack

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Frontend | React + Next.js | 18.x / 15.x | Schema editor, data preview grid |
| Data Grid | AG Grid | 32.x | High-performance tabular data preview |
| Styling | Tailwind CSS | 3.x | Responsive layout |
| API Framework | FastAPI | 0.115+ | Async REST endpoints, streaming exports |
| Agent Framework | LangChain | 0.3+ | LLM-assisted data generation |
| Data Generation | Faker + Mimesis | 30.x / 18.x | Locale-aware realistic values |
| PII Detection | Presidio Analyzer + Anonymiser | 2.x | PII scanning & redaction |
| LLM Providers | Claude, Gemini | Latest | Edge-case reasoning, schema understanding |
| Database | PostgreSQL | 16 | Metadata store |
| Object Storage | Azure Blob / S3 | — | Dataset file storage |
| Serialisation | Apache Arrow / Parquet | — | Columnar export format |
| Cache / Queue | Redis + Celery | 7.x / 5.x | Async generation jobs |
| Containerisation | Docker | 24+ | Packaging |
| Orchestration | Kubernetes (AKS) | 1.29+ | Deployment |

---

## 4. API Contracts

### 4.1 Import Schema

```
POST /api/v1/datageni/schemas/import
Content-Type: application/json

Request Body:
{
  "source": "ddl" | "json_schema" | "db_introspect",
  "ddl": "CREATE TABLE users (id INT PRIMARY KEY, ...)",  // if source=ddl
  "json_schema": { ... },  // if source=json_schema
  "connection_string": "postgresql://...",  // if source=db_introspect
  "tables": ["users", "orders"]  // filter tables (db_introspect)
}

Response 201 Created:
{
  "schema_id": "uuid",
  "tables": [
    {
      "name": "users",
      "columns": [
        { "name": "id", "type": "integer", "pk": true, "nullable": false },
        { "name": "email", "type": "varchar(255)", "pii": true },
        { "name": "name", "type": "varchar(100)", "pii": true }
      ],
      "foreign_keys": []
    }
  ],
  "pii_columns_detected": ["users.email", "users.name"]
}
```

### 4.2 Generate Dataset

```
POST /api/v1/datageni/datasets/generate
Content-Type: application/json

Request Body:
{
  "schema_id": "uuid",
  "config": {
    "row_count": 10000,
    "locale": "en_GB",
    "edge_case_pct": 15,
    "null_pct": 5,
    "pii_mode": "synthetic" | "redacted" | "masked",
    "seed": 42
  },
  "tables": ["users", "orders"],
  "model_preference": "claude" | "gemini" | "auto"
}

Response 202 Accepted:
{
  "job_id": "uuid",
  "status": "generating",
  "estimated_seconds": 45
}
```

### 4.3 Get Job Status / Download

```
GET /api/v1/datageni/jobs/{job_id}

Response 200 OK:
{
  "job_id": "uuid",
  "status": "completed",
  "dataset_id": "uuid",
  "row_counts": { "users": 10000, "orders": 35000 },
  "fk_violations": 0,
  "pii_clean": true,
  "generation_time_ms": 38000,
  "download_urls": {
    "csv": "/api/v1/datageni/datasets/{id}/download?format=csv",
    "json": "/api/v1/datageni/datasets/{id}/download?format=json",
    "sql": "/api/v1/datageni/datasets/{id}/download?format=sql",
    "parquet": "/api/v1/datageni/datasets/{id}/download?format=parquet"
  }
}
```

### 4.4 Export to Auto-PlayPilot / Perf-Xi

```
POST /api/v1/datageni/datasets/{dataset_id}/export
Content-Type: application/json

Request Body:
{
  "target": "auto-playpilot" | "perf-xi",
  "format": "json_fixtures" | "csv_load",
  "tables": ["users", "orders"]
}

Response 200 OK:
{
  "export_id": "uuid",
  "target": "auto-playpilot",
  "status": "delivered",
  "rows_exported": 10000
}
```

### 4.5 Preview (Sampled)

```
GET /api/v1/datageni/datasets/{dataset_id}/preview?table=users&limit=50

Response 200 OK:
{
  "table": "users",
  "total_rows": 10000,
  "sample": [
    { "id": 1, "email": "jane.doe@example.com", "name": "Jane Doe" },
    ...
  ]
}
```

---

## 5. Data Model

```mermaid
erDiagram
    SCHEMA_DEF {
        uuid id PK
        string name
        json tables "table definitions + columns"
        json foreign_keys
        string source "ddl | json_schema | db_introspect"
        json pii_columns
        uuid project_id FK
        timestamp created_at
    }

    DATASET {
        uuid id PK
        uuid schema_id FK
        json config "row_count, locale, edge_pct, seed"
        string status "generating | completed | failed"
        json row_counts "per-table counts"
        int fk_violations
        boolean pii_clean
        string blob_path "azure://container/path"
        int generation_time_ms
        timestamp created_at
    }

    GENERATION_JOB {
        uuid id PK
        uuid dataset_id FK
        string status "queued | generating | completed | failed"
        string model_used
        json progress "per-table progress %"
        timestamp started_at
        timestamp completed_at
    }

    EXPORT_RECORD {
        uuid id PK
        uuid dataset_id FK
        string target "auto-playpilot | perf-xi | download"
        string format "csv | json | sql | parquet"
        int rows_exported
        timestamp created_at
    }

    SEED_LIBRARY {
        uuid id PK
        string name "UK addresses | credit card patterns"
        string locale
        json seed_data
        string category "address | finance | healthcare"
    }

    SCHEMA_DEF ||--o{ DATASET : "generates"
    DATASET ||--|| GENERATION_JOB : "tracked by"
    DATASET ||--o{ EXPORT_RECORD : "exported via"
```

### Key Tables

| Table | Rows (est.) | Indexes |
|---|---|---|
| `schema_def` | ~500 per tenant | `(project_id)`, `(source)` |
| `dataset` | ~5 K per tenant | `(schema_id, status)`, `(created_at)` |
| `generation_job` | ~5 K per tenant | `(status)`, `(dataset_id)` |
| `export_record` | ~10 K per tenant | `(dataset_id, target)` |
| `seed_library` | ~200 global | `(category, locale)` |

---

## 6. Integration Patterns

### 6.1 Upstream Integrations

| Source | Protocol | Details |
|---|---|---|
| **CaseGeni** | REST API (internal) | Receives data needs per test case; triggers schema-aware generation |
| **DB Introspection** | JDBC / psycopg2 | Connects to target DB; reads information_schema for table definitions |

### 6.2 Downstream Integrations

| Target | Protocol | Trigger | Payload |
|---|---|---|---|
| **Auto-PlayPilot** | REST API (internal) | Manual or auto-export | JSON fixtures for Playwright test setup |
| **Perf-Xi** | REST API (internal) | Manual export | CSV/Parquet bulk load data for k6 scenarios |
| **Insights360** | Event bus (Redis pub/sub) | On generation complete | `{ dataset_id, rows, tables, pii_clean }` |

### 6.3 Data Generation Pipeline

```mermaid
flowchart TB
    SCHEMA[Schema Definition] --> PLAN[Generation Plan<br/>Topological sort by FK]
    PLAN --> TABLE1[Generate Parent Tables<br/>Faker / Mimesis + LLM edge cases]
    TABLE1 --> FK[Resolve FK References<br/>Link child → parent PKs]
    FK --> TABLE2[Generate Child Tables]
    TABLE2 --> PII[PII Scan<br/>Presidio Analyzer]
    PII --> CHECK{PII Found?}
    CHECK -->|Yes| ANON[Anonymise / Mask<br/>Presidio Anonymiser]
    CHECK -->|No| VALID[Validate Constraints]
    ANON --> VALID
    VALID --> STORE[Store to Blob<br/>CSV / Parquet]
```

---

## 7. Deployment Configuration

### 7.1 Kubernetes Resources

```yaml
# datageni-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: datageni-api
  namespace: zenseai-qi
spec:
  replicas: 3
  selector:
    matchLabels:
      app: datageni-api
  template:
    metadata:
      labels:
        app: datageni-api
    spec:
      containers:
        - name: datageni-api
          image: zenseai.azurecr.io/datageni-api:latest
          ports:
            - containerPort: 8002
          resources:
            requests:
              cpu: "1000m"
              memory: "1Gi"
            limits:
              cpu: "4000m"
              memory: "4Gi"
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: datageni-secrets
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                secretKeyRef:
                  name: datageni-secrets
                  key: redis-url
            - name: BLOB_STORAGE_URL
              valueFrom:
                secretKeyRef:
                  name: datageni-secrets
                  key: blob-storage-url
            - name: LLM_API_KEY
              valueFrom:
                secretKeyRef:
                  name: datageni-secrets
                  key: llm-api-key
          livenessProbe:
            httpGet:
              path: /health
              port: 8002
            initialDelaySeconds: 10
            periodSeconds: 30
---
# Celery worker for async generation
apiVersion: apps/v1
kind: Deployment
metadata:
  name: datageni-worker
  namespace: zenseai-qi
spec:
  replicas: 5
  selector:
    matchLabels:
      app: datageni-worker
  template:
    metadata:
      labels:
        app: datageni-worker
    spec:
      containers:
        - name: worker
          image: zenseai.azurecr.io/datageni-api:latest
          command: ["celery", "-A", "app.worker", "worker", "--concurrency=4"]
          resources:
            requests:
              cpu: "1000m"
              memory: "2Gi"
            limits:
              cpu: "4000m"
              memory: "8Gi"
```

### 7.2 Environment Variables

| Variable | Description | Example |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection | `postgresql://user:pass@host:5432/datageni` |
| `REDIS_URL` | Redis (queue + cache) | `redis://host:6379/2` |
| `BLOB_STORAGE_URL` | Azure Blob connection string | `DefaultEndpointsProtocol=https;...` |
| `LLM_API_KEY` | Claude / Gemini API key | `key-...` |
| `MAX_ROWS` | Maximum rows per dataset | `1000000` |
| `PII_SCAN_ENABLED` | Toggle Presidio scanning | `true` |
| `CELERY_CONCURRENCY` | Worker parallelism | `4` |
| `LOG_LEVEL` | Log verbosity | `INFO` |

### 7.3 Scaling Policy

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: datageni-worker-hpa
  namespace: zenseai-qi
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: datageni-worker
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 65
```

---

## 8. Folder Structure

```
datageni/
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── schema/             # Schema designer
│   │   │   ├── generate/           # Config & launch
│   │   │   ├── preview/            # Data preview grid
│   │   │   └── export/             # Export manager
│   │   ├── components/
│   │   │   ├── SchemaEditor.tsx
│   │   │   ├── ColumnConfig.tsx
│   │   │   ├── DataGrid.tsx        # AG Grid wrapper
│   │   │   └── ExportDialog.tsx
│   │   ├── hooks/
│   │   │   ├── useSchema.ts
│   │   │   └── useGeneration.ts
│   │   └── lib/
│   │       ├── api-client.ts
│   │       └── types.ts
│   ├── package.json
│   └── tsconfig.json
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── worker.py               # Celery worker entrypoint
│   │   ├── routers/
│   │   │   ├── schemas.py          # Schema CRUD + import
│   │   │   ├── datasets.py         # Generation + download
│   │   │   └── export.py           # Downstream push
│   │   ├── services/
│   │   │   ├── schema_parser.py    # DDL, JSON Schema, DB introspect
│   │   │   ├── generator.py        # Faker/Mimesis orchestration
│   │   │   ├── fk_resolver.py      # Topological sort + FK linking
│   │   │   ├── pii_scanner.py      # Presidio integration
│   │   │   ├── agent_client.py     # Test Data Generator calls
│   │   │   └── exporter.py         # CSV, JSON, SQL, Parquet writers
│   │   ├── models/
│   │   │   ├── schema_def.py
│   │   │   └── dataset.py
│   │   ├── schemas/
│   │   │   ├── schema_def.py       # Pydantic models
│   │   │   └── dataset.py
│   │   └── config.py
│   ├── seed_library/               # Pre-built locale-aware seeds
│   │   ├── uk_addresses.json
│   │   ├── us_ssn_patterns.json
│   │   └── healthcare_codes.json
│   ├── alembic/
│   ├── tests/
│   │   ├── test_generator.py
│   │   ├── test_fk_resolver.py
│   │   ├── test_pii_scanner.py
│   │   └── test_exporter.py
│   ├── requirements.txt
│   └── Dockerfile
├── k8s/
│   ├── api-deployment.yaml
│   ├── worker-deployment.yaml
│   ├── service.yaml
│   ├── hpa.yaml
│   └── secrets.yaml
└── docker-compose.yaml
```

---

## 9. Security Considerations

| Concern | Mitigation |
|---|---|
| **PII in generated data** | Presidio scans every dataset; PII mode: synthetic (fake values), redacted, or masked |
| **DB connection strings** | Encrypted in Azure Key Vault; read-only introspection user; connection pooling with timeout |
| **Production data exposure** | Never copy production data — generate synthetic equivalents; schema-only introspection |
| **Authentication** | Entra ID OAuth 2.0 with JWT; per-tenant dataset isolation |
| **Authorisation** | Roles: Schema Designer, Generator, Exporter, Admin |
| **Blob storage** | Private containers; SAS tokens with expiry for downloads |
| **LLM data leakage** | Only schema metadata sent to LLM, never actual data rows |

---

## 10. Performance Targets

| Metric | Target | Notes |
|---|---|---|
| 10 K rows (single table) | ≤ 15 s | Faker + edge-case injection |
| 100 K rows (multi-table) | ≤ 120 s | Parallelised per table, FK resolution |
| 1 M rows | ≤ 10 min | Celery workers, streaming Parquet |
| FK validation (100 K rows) | ≤ 5 s | In-memory hash join |
| PII scan (100 K rows) | ≤ 30 s | Batch Presidio analysis |
| Export to Parquet (1 M rows) | ≤ 20 s | Apache Arrow columnar write |
| API throughput | ≥ 100 req/s | At 3 API pods |
| Availability | 99.9 % | Multi-AZ deployment |

---

## 11. Implementation Roadmap

| Phase | Timeline | Deliverables |
|---|---|---|
| **Phase 1 — Core** | Weeks 1–2 | Schema import (DDL + JSON Schema), basic Faker generation, CSV export |
| **Phase 2 — FK & Edge** | Weeks 3–4 | FK resolver, edge-case injection, multi-table generation |
| **Phase 3 — PII & LLM** | Weeks 5–6 | Presidio integration, LLM-assisted edge cases, seed library |
| **Phase 4 — Integrations** | Weeks 7–8 | CaseGeni ingest, Auto-PlayPilot + Perf-Xi export, DB introspection |
| **Phase 5 — Scale** | Weeks 9–10 | Parquet export, 1 M row support, Celery worker scaling, UI polish |

---

📄 [Full Requirements Spec →](../solutions/datageni.md) · 🤖 [Agent Design →](../agents/test-data-generator.md) · [← Platform Architecture](../architecture.md)
