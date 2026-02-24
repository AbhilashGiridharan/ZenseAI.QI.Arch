---
layout: default
title: "Perf-Xi — Solution Architecture & Implementation"
description: "Detailed architecture, component design, API contracts, data model, and implementation guide for Perf-Xi."
---

[← Back to Platform Architecture](../architecture.md) · [Requirements Spec](../solutions/perf-xi.md) · [Agent Design](../agents/performance-agent.md)
{: .fs-3 }

# Perf-Xi — Solution Architecture & Implementation

---

## 1. Architecture Overview

<img src="../../assets/images/arch-perf-xi.svg" alt="Perf-Xi Architecture" width="100%" style="border-radius:6px; margin-bottom:1rem;" />

```mermaid
flowchart LR
    SPEC[OpenAPI Specs<br/>User Flows] --> UI

    subgraph Users
        PE([Performance Engineer])
        TM([Technical Manager])
    end

    subgraph PerfXi["Perf-Xi"]
        UI[Perf-Xi UI<br/>Scripts · SLOs · Profiles]
        API[Perf-Xi API]
    end

    subgraph Agent
        PA[Performance Agent<br/>k6 Gen · SLO · Capacity]
    end

    subgraph Runners
        K6L[k6 Local]
        K6C[k6 Cloud]
    end

    subgraph Models["Model Providers"]
        CLA[Claude]
        GEM[Gemini]
    end

    DB[(PostgreSQL<br/>SLIs · SLOs · Runs)]

    Users --> UI --> API --> PA
    PA --> CLA & GEM
    PA --> K6L & K6C
    PA -->|SLI metrics| API
    API --> DB

    API -.->|SLI data| I3[Insights360]
    API -.->|breach| CI[CI Gate ❌]
```

### Key Flows

- k6 scripts auto-generated from OpenAPI specs with auth patterns
- SLIs (p50, p95, p99, error rate) defined per endpoint; SLO thresholds enforced
- Load profiles: ramp, soak, spike — configurable per scenario
- SLO breaches trigger CI pipeline fail signals
- AI regression analysis + capacity-planning reports
- SLI data published to **Insights360** dashboard gauges

---

## 2. Component Breakdown

```mermaid
flowchart TB
    subgraph FrontEnd["Perf-Xi UI (React / Next.js)"]
        SCRIPTS[Script Manager<br/>k6 scripts · OpenAPI import]
        SLO_UI[SLO Configurator<br/>Thresholds · Budgets]
        PROFILE[Load Profile Designer<br/>Ramp · Soak · Spike]
        RESULTS[Results Dashboard<br/>SLI charts · Percentiles]
        CAPACITY[Capacity Planner<br/>AI projections]
    end

    subgraph APILayer["Perf-Xi API (FastAPI)"]
        GENSVC[Script Generation<br/>Agent orchestration]
        RUNSVC[Execution Service<br/>k6 local + cloud dispatch]
        SLISVC[SLI Collector<br/>Metrics aggregation]
        SLOSVC[SLO Evaluator<br/>Threshold checks · CI gate]
        CAPSVC[Capacity Service<br/>AI regression analysis]
    end

    subgraph AgentLayer["Performance Agent"]
        K6GEN[k6 Script Generator]
        SLOGEN[SLO Recommender]
        REGR[Regression Analyser]
        CAP[Capacity Planner]
    end

    subgraph DataLayer["Data Stores"]
        PG[(PostgreSQL<br/>Scripts · SLOs · Runs)]
        TS[(TimescaleDB<br/>SLI time-series)]
        REDIS[(Redis<br/>Job queue · Live metrics)]
    end

    FrontEnd --> APILayer
    APILayer --> AgentLayer
    APILayer --> DataLayer
```

| Component | Technology | Responsibility |
|---|---|---|
| **Perf-Xi UI** | React 18 / Next.js 15, Recharts, Tailwind | Script management, SLO config, load profiles, results |
| **Perf-Xi API** | Python 3.12, FastAPI, Uvicorn | Script gen, execution dispatch, SLI collection, SLO evaluation |
| **Performance Agent** | Python 3.12, LangChain/LangGraph | k6 generation, SLO recommendation, regression analysis |
| **k6 Runner** | k6 OSS / k6 Cloud | Load test execution, metrics collection |
| **PostgreSQL + TimescaleDB** | PostgreSQL 16 + TimescaleDB | Scripts, SLOs, SLI time-series |
| **Redis** | Redis 7 | Job queue, live metric streaming |

---

## 3. Tech Stack

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Frontend | React + Next.js | 18.x / 15.x | Script IDE, SLO config, results dashboards |
| Charts | Recharts + D3.js | 2.x / 7.x | SLI percentile charts, load curves |
| Code Editor | Monaco Editor | 0.50+ | k6 JavaScript script editing |
| API Framework | FastAPI | 0.115+ | Async REST endpoints |
| Agent Framework | LangChain / LangGraph | 0.3+ | k6 generation, capacity planning |
| LLM Providers | Claude, Gemini | Latest | Script gen, regression analysis |
| Load Testing | k6 (Grafana) | 0.53+ | Load test runner |
| k6 Cloud | Grafana Cloud k6 | — | Distributed load generation |
| Database | PostgreSQL + TimescaleDB | 16 / 2.x | Metadata + time-series |
| Cache / Queue | Redis | 7.x | Job queue, live SLI streaming |
| Observability | Prometheus + Grafana | — | k6 metrics visualisation |
| Containerisation | Docker | 24+ | Packaging |
| Orchestration | Kubernetes (AKS) | 1.29+ | Deployment |

---

## 4. API Contracts

### 4.1 Generate k6 Script from OpenAPI

```
POST /api/v1/perfxi/scripts/generate
Content-Type: application/json

Request Body:
{
  "openapi_url": "https://api.app.com/openapi.json",
  "endpoints": ["/api/users", "/api/orders"],
  "auth_pattern": "bearer" | "cookie" | "api_key" | "none",
  "load_profile": "ramp" | "soak" | "spike",
  "vus": 100,
  "duration": "5m",
  "model_preference": "claude" | "gemini" | "auto"
}

Response 202 Accepted:
{
  "job_id": "uuid",
  "status": "generating",
  "estimated_seconds": 15
}
```

### 4.2 Get Generated Script

```
GET /api/v1/perfxi/scripts/{script_id}

Response 200 OK:
{
  "script_id": "uuid",
  "filename": "api-load-test.js",
  "code": "import http from 'k6/http';\nimport { check } from 'k6';\n...",
  "endpoints": ["/api/users", "/api/orders"],
  "load_profile": {
    "type": "ramp",
    "stages": [
      { "duration": "1m", "target": 20 },
      { "duration": "3m", "target": 100 },
      { "duration": "1m", "target": 0 }
    ]
  },
  "model_used": "claude-sonnet-4-20250514"
}
```

### 4.3 Configure SLOs

```
POST /api/v1/perfxi/slos
Content-Type: application/json

Request Body:
{
  "project_id": "uuid",
  "slos": [
    {
      "endpoint": "/api/users",
      "method": "GET",
      "thresholds": {
        "p50_ms": 100,
        "p95_ms": 300,
        "p99_ms": 500,
        "error_rate_pct": 0.1
      },
      "budget_window": "30d",
      "budget_pct": 99.9
    }
  ]
}

Response 201 Created:
{
  "slo_set_id": "uuid",
  "slos_created": 1
}
```

### 4.4 Execute Load Test

```
POST /api/v1/perfxi/runs
Content-Type: application/json

Request Body:
{
  "script_id": "uuid",
  "runner": "local" | "cloud",
  "environment": "staging" | "qa",
  "data_source": {
    "datageni_dataset_id": "uuid",
    "format": "csv"
  },
  "slo_set_id": "uuid"
}

Response 202 Accepted:
{
  "run_id": "uuid",
  "status": "running",
  "estimated_duration": "5m"
}
```

### 4.5 Get Run Results

```
GET /api/v1/perfxi/runs/{run_id}

Response 200 OK:
{
  "run_id": "uuid",
  "status": "completed",
  "summary": {
    "vus_max": 100,
    "duration_ms": 300000,
    "requests_total": 45000,
    "rps_avg": 150
  },
  "slis": [
    {
      "endpoint": "/api/users",
      "method": "GET",
      "p50_ms": 85,
      "p95_ms": 245,
      "p99_ms": 420,
      "error_rate_pct": 0.02,
      "slo_status": "pass"
    }
  ],
  "slo_breaches": [],
  "ci_gate": "pass"
}
```

### 4.6 AI Capacity Planning

```
POST /api/v1/perfxi/capacity/analyse
Content-Type: application/json

Request Body:
{
  "project_id": "uuid",
  "target_rps": 500,
  "target_p95_ms": 300,
  "historical_runs": 10
}

Response 200 OK:
{
  "analysis": {
    "current_capacity_rps": 180,
    "target_rps": 500,
    "scaling_factor": 2.8,
    "recommendations": [
      "Scale API pods from 3 to 8",
      "Add read replica for PostgreSQL",
      "Enable Redis caching for /api/users"
    ],
    "estimated_infra_cost_delta": "+$420/month",
    "confidence": 0.82
  },
  "model_used": "gemini-2.5-pro"
}
```

---

## 5. Data Model

```mermaid
erDiagram
    SCRIPT {
        uuid id PK
        string filename
        text code
        json endpoints
        json load_profile
        string model_used
        uuid project_id FK
        timestamp created_at
    }

    SLO_SET {
        uuid id PK
        uuid project_id FK
        json slos "endpoint thresholds"
        string budget_window "30d"
        float budget_pct "99.9"
        timestamp created_at
    }

    RUN {
        uuid id PK
        uuid script_id FK
        uuid slo_set_id FK
        string runner "local | cloud"
        string environment
        string status "running | completed | failed"
        int vus_max
        int duration_ms
        int requests_total
        float rps_avg
        string ci_gate "pass | fail"
        timestamp started_at
        timestamp completed_at
    }

    SLI_METRIC {
        uuid id PK
        uuid run_id FK
        string endpoint
        string method
        float p50_ms
        float p95_ms
        float p99_ms
        float error_rate_pct
        string slo_status "pass | fail"
        timestamp recorded_at
    }

    CAPACITY_REPORT {
        uuid id PK
        uuid project_id FK
        float current_rps
        float target_rps
        json recommendations
        float confidence
        string model_used
        timestamp generated_at
    }

    SCRIPT ||--o{ RUN : "executed as"
    SLO_SET ||--o{ RUN : "evaluated against"
    RUN ||--o{ SLI_METRIC : "produces"
    PROJECT ||--o{ SCRIPT : "contains"
    PROJECT ||--o{ SLO_SET : "defines"
    PROJECT ||--o{ CAPACITY_REPORT : "analysed by"
```

### Key Tables

| Table | Rows (est.) | Indexes | Partitioning |
|---|---|---|---|
| `sli_metric` | ~500 K per tenant/month | `(run_id, endpoint)`, `(recorded_at)` | TimescaleDB hypertable |
| `run` | ~5 K per tenant | `(script_id, status)`, `(started_at)` | — |
| `script` | ~500 per tenant | `(project_id)` | — |
| `slo_set` | ~50 per tenant | `(project_id)` | — |

---

## 6. Integration Patterns

### 6.1 Upstream Integrations

| Source | Protocol | Details |
|---|---|---|
| **OpenAPI Specs** | HTTP fetch | Pull spec from target API URL; parse endpoints |
| **DataGeni** | REST API (internal) | Load test data (CSV) for parameterised k6 scenarios |

### 6.2 Downstream Integrations

| Target | Protocol | Trigger | Payload |
|---|---|---|---|
| **Insights360** | Event bus (Redis pub/sub) | On run complete | `{ run_id, slis[], slo_status, ci_gate }` |
| **CI Pipeline** | Webhook / REST | On SLO breach | `{ status: "fail", breaches: [...] }` |
| **Grafana** | Prometheus remote write | During run | k6 metrics in real-time |

### 6.3 Load Test Execution Flow

```mermaid
flowchart TB
    TRIGGER[API / CI Trigger] --> DISPATCH[Dispatch to Runner]
    DISPATCH --> LOCAL{Local?}
    LOCAL -->|Yes| K6L[k6 Local<br/>Docker container]
    LOCAL -->|No| K6C[k6 Cloud<br/>Distributed]
    K6L --> METRICS[Metrics Stream<br/>Prometheus + Redis]
    K6C --> METRICS
    METRICS --> COLLECT[SLI Collector<br/>p50, p95, p99, errors]
    COLLECT --> EVAL[SLO Evaluator<br/>Threshold check]
    EVAL --> PASS{SLO Pass?}
    PASS -->|Yes| REPORT[Store Results<br/>Publish to Insights360]
    PASS -->|No| BREACH[SLO Breach Alert<br/>CI Gate FAIL]
    BREACH --> REPORT
```

---

## 7. Deployment Configuration

### 7.1 Kubernetes Resources

```yaml
# perf-xi-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: perfxi-api
  namespace: zenseai-qi
spec:
  replicas: 2
  selector:
    matchLabels:
      app: perfxi-api
  template:
    metadata:
      labels:
        app: perfxi-api
    spec:
      containers:
        - name: perfxi-api
          image: zenseai.azurecr.io/perfxi-api:latest
          ports:
            - containerPort: 8004
          resources:
            requests:
              cpu: "500m"
              memory: "512Mi"
            limits:
              cpu: "2000m"
              memory: "2Gi"
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: perfxi-secrets
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                secretKeyRef:
                  name: perfxi-secrets
                  key: redis-url
            - name: LLM_API_KEY
              valueFrom:
                secretKeyRef:
                  name: perfxi-secrets
                  key: llm-api-key
            - name: K6_CLOUD_TOKEN
              valueFrom:
                secretKeyRef:
                  name: perfxi-secrets
                  key: k6-cloud-token
---
# k6 runner pods (ephemeral, job-based)
apiVersion: batch/v1
kind: Job
metadata:
  name: k6-run-template
  namespace: zenseai-qi
spec:
  template:
    spec:
      containers:
        - name: k6
          image: grafana/k6:0.53.0
          command: ["k6", "run", "--out", "experimental-prometheus-rw", "/scripts/test.js"]
          resources:
            requests:
              cpu: "2000m"
              memory: "2Gi"
            limits:
              cpu: "4000m"
              memory: "4Gi"
          volumeMounts:
            - name: scripts
              mountPath: /scripts
      volumes:
        - name: scripts
          configMap:
            name: k6-script-cm
      restartPolicy: Never
```

### 7.2 Environment Variables

| Variable | Description | Example |
|---|---|---|
| `DATABASE_URL` | PostgreSQL + TimescaleDB | `postgresql://user:pass@host:5432/perfxi` |
| `REDIS_URL` | Redis (queue + live metrics) | `redis://host:6379/4` |
| `LLM_API_KEY` | Claude / Gemini key | `key-...` |
| `K6_CLOUD_TOKEN` | Grafana Cloud k6 API token | `token-...` |
| `PROMETHEUS_REMOTE_WRITE_URL` | Prometheus endpoint | `http://prometheus:9090/api/v1/write` |
| `CI_WEBHOOK_URL` | CI pipeline callback | `https://ci.org/hooks/perfxi` |
| `LOG_LEVEL` | Log verbosity | `INFO` |

---

## 8. Folder Structure

```
perf-xi/
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── scripts/            # k6 script manager
│   │   │   ├── slos/               # SLO configurator
│   │   │   ├── profiles/           # Load profile designer
│   │   │   ├── runs/               # Execution & results
│   │   │   └── capacity/           # AI capacity planner
│   │   ├── components/
│   │   │   ├── K6Editor.tsx        # Monaco k6 editor
│   │   │   ├── SLIChart.tsx        # Percentile charts
│   │   │   ├── LoadCurve.tsx       # VU ramp visualisation
│   │   │   ├── SLOBadge.tsx
│   │   │   └── CapacityReport.tsx
│   │   ├── hooks/
│   │   │   ├── useRuns.ts
│   │   │   └── useSLOs.ts
│   │   └── lib/
│   │       ├── api-client.ts
│   │       └── types.ts
│   ├── package.json
│   └── tsconfig.json
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── routers/
│   │   │   ├── scripts.py          # k6 script CRUD + generate
│   │   │   ├── slos.py             # SLO configuration
│   │   │   ├── runs.py             # Execution dispatch + results
│   │   │   └── capacity.py         # AI capacity analysis
│   │   ├── services/
│   │   │   ├── agent_client.py     # Performance Agent calls
│   │   │   ├── k6_runner.py        # k6 local/cloud dispatch
│   │   │   ├── sli_collector.py    # Metrics aggregation
│   │   │   ├── slo_evaluator.py    # Threshold enforcement
│   │   │   └── ci_gate.py          # CI pipeline integration
│   │   ├── models/
│   │   │   ├── script.py
│   │   │   ├── slo.py
│   │   │   └── run.py
│   │   ├── schemas/
│   │   │   ├── script.py
│   │   │   └── run.py
│   │   └── config.py
│   ├── alembic/
│   ├── tests/
│   │   ├── test_k6_gen.py
│   │   ├── test_slo_eval.py
│   │   └── test_capacity.py
│   ├── requirements.txt
│   └── Dockerfile
├── agent/
│   ├── performance_agent/
│   │   ├── chain.py                # k6 gen + capacity chain
│   │   ├── prompts/
│   │   │   ├── k6_generate.txt
│   │   │   ├── slo_recommend.txt
│   │   │   └── capacity_plan.txt
│   │   └── router.py
│   └── tests/
│       └── test_perf_agent.py
├── k8s/
│   ├── api-deployment.yaml
│   ├── k6-job-template.yaml
│   ├── service.yaml
│   └── secrets.yaml
└── docker-compose.yaml
```

---

## 9. Security Considerations

| Concern | Mitigation |
|---|---|
| **Target API credentials** | Auth tokens stored in Key Vault; injected via k6 env vars at runtime |
| **k6 Cloud tokens** | Encrypted in Kubernetes secrets; scoped to specific projects |
| **Load test impact** | Staging/QA environments only; rate-limiting safeguards; max VUs cap |
| **Authentication** | Entra ID OAuth 2.0; RBAC per tenant |
| **Authorisation** | Roles: Script Author, Runner, SLO Admin, Admin |
| **Results data** | Tenant-isolated; no cross-project leakage |
| **CI gate integrity** | Signed webhook payloads; replay protection |

---

## 10. Performance Targets

| Metric | Target | Notes |
|---|---|---|
| k6 script generation | ≤ 12 s | Including LLM round-trip |
| SLI collection (per run) | ≤ 2 s after run completes | Prometheus query + aggregate |
| SLO evaluation | ≤ 500 ms | In-memory threshold check |
| Capacity analysis | ≤ 20 s | LLM + historical regression |
| Dashboard load | ≤ 2 s | Redis-cached SLI snapshots |
| k6 runner spin-up | ≤ 30 s | Pod scheduling + script mount |
| API throughput | ≥ 100 req/s | At 2 API pods |
| Availability | 99.9 % | Multi-AZ deployment |

---

## 11. Implementation Roadmap

| Phase | Timeline | Deliverables |
|---|---|---|
| **Phase 1 — Core** | Weeks 1–2 | k6 script CRUD, OpenAPI import, basic generation |
| **Phase 2 — SLO** | Weeks 3–4 | SLO configurator, SLI collection, threshold evaluation |
| **Phase 3 — Execution** | Weeks 5–6 | k6 local runner, k6 Cloud integration, results dashboard |
| **Phase 4 — CI & AI** | Weeks 7–8 | CI gate integration, AI regression analysis, capacity planner |
| **Phase 5 — Polish** | Weeks 9–10 | Load profile designer, Grafana integration, performance tuning |

---

📄 [Full Requirements Spec →](../solutions/perf-xi.md) · 🤖 [Agent Design →](../agents/performance-agent.md) · [← Platform Architecture](../architecture.md)
