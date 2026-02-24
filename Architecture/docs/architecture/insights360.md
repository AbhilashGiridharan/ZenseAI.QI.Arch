---
layout: default
title: "Insights360 — Solution Architecture & Implementation"
description: "Detailed architecture, component design, API contracts, data model, and implementation guide for Insights360."
---

[← Back to Platform Architecture](../architecture.md) · [Requirements Spec](../solutions/insights360.md) · [Agent Design](../agents/report-agent.md)
{: .fs-3 }

# Insights360 — Solution Architecture & Implementation

---

## 1. Architecture Overview

<img src="../../assets/images/arch-insights360.svg" alt="Insights360 Architecture" width="100%" style="border-radius:6px; margin-bottom:1rem;" />

```mermaid
flowchart LR
    subgraph Sources["Data Sources"]
        APP[Auto-PlayPilot]
        CG[CaseGeni]
        DG[DataGeni]
        PX[Perf-Xi]
        SX[Secure-Xi]
        DS[DeepSpeci]
    end

    subgraph Users
        MGR([Manager])
        TM([Technical Manager])
        NTU([Non-Technical User])
    end

    subgraph Insights360
        API[Insights360 API<br/>Aggregate · Transform]
        DASH[Dashboard UI<br/>KPIs · Heatmap · Trends]
    end

    subgraph Agent
        RA[Report Agent<br/>AI Narrative · Digest]
    end

    subgraph Models["Model Providers"]
        GEM[Gemini]
        CLA[Claude]
    end

    subgraph Outputs
        PDF[PDF / CSV]
        SLACK[Slack / Teams]
        EMAIL[Email Digest]
    end

    DB[(PostgreSQL + Redis<br/>Metrics · Cache)]

    Sources --> API
    Users --> DASH
    API --> RA --> GEM & CLA
    RA --> API
    API --> DASH
    API --> DB
    DASH --> PDF & SLACK & EMAIL
```

### Key Flows

- Aggregates data from **all 6 sibling solutions** into a unified view
- The **Report Agent** generates AI release-readiness narratives (≤ 300 words)
- Dashboard provides drill-down from KPI tiles → individual test results
- Exports: PDF, CSV; notifications: Slack, Teams, email digest (daily 08:00)
- Performance SLI gauges from Perf-Xi + security posture badge from Secure-Xi overlay

---

## 2. Component Breakdown

```mermaid
flowchart TB
    subgraph FrontEnd["Insights360 UI (React / Next.js)"]
        KPI[KPI Tiles<br/>Pass rate · Coverage · SLIs]
        HEAT[Coverage Heatmap<br/>Req ↔ Test matrix]
        TREND[Trend Charts<br/>Time-series · Sparklines]
        DRILL[Drill-Down Views<br/>Per solution detail]
        REPORT[Report Viewer<br/>AI narrative · PDF]
    end

    subgraph APILayer["Insights360 API (FastAPI)"]
        AGG[Aggregation Service<br/>Cross-solution metrics]
        TRANS[Transform Service<br/>KPI computation]
        NOTIFY[Notification Service<br/>Slack · Teams · Email]
        EXPSVC[Export Service<br/>PDF · CSV generation]
        NARR[Narrative Service<br/>Report Agent orchestration]
    end

    subgraph AgentLayer["Report Agent"]
        READINESS[Release Readiness Assessor]
        SUMMARY[Executive Summary Generator]
        ANOMALY[Anomaly Detector<br/>Regression alerts]
    end

    subgraph DataLayer["Data Stores"]
        PG[(PostgreSQL<br/>Metrics · KPIs · Reports)]
        REDIS[(Redis<br/>Real-time cache · Pub/Sub)]
        TS[(TimescaleDB ext.<br/>Time-series metrics)]
    end

    FrontEnd --> APILayer
    APILayer --> AgentLayer
    APILayer --> DataLayer
```

| Component | Technology | Responsibility |
|---|---|---|
| **Insights360 UI** | React 18 / Next.js 15, Recharts, Tailwind | KPI tiles, heatmaps, trends, drill-downs, report viewer |
| **Insights360 API** | Python 3.12, FastAPI, Uvicorn | Aggregation, KPI computation, notifications, export |
| **Report Agent** | Python 3.12, LangChain/LangGraph | AI narrative generation, anomaly detection |
| **PostgreSQL + TimescaleDB** | PostgreSQL 16 + TimescaleDB | Metrics storage, time-series queries |
| **Redis** | Redis 7 | Real-time event ingestion, dashboard caching |

---

## 3. Tech Stack

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Frontend | React + Next.js | 18.x / 15.x | Interactive dashboard SPA |
| Charts | Recharts + D3.js | 2.x / 7.x | KPI tiles, sparklines, heatmaps |
| Styling | Tailwind CSS | 3.x | Responsive layout |
| API Framework | FastAPI | 0.115+ | Async aggregation endpoints |
| Agent Framework | LangChain / LangGraph | 0.3+ | AI narrative orchestration |
| LLM Providers | Gemini, Claude | Latest | Report generation, anomaly reasoning |
| Database | PostgreSQL + TimescaleDB | 16 / 2.x | Metrics & time-series store |
| Cache / Events | Redis | 7.x | Pub/sub event bus, dashboard cache |
| PDF Generation | WeasyPrint | 62+ | PDF report export |
| Notifications | Slack SDK, MS Graph, SMTP | — | Multi-channel alerts |
| Containerisation | Docker | 24+ | Packaging |
| Orchestration | Kubernetes (AKS) | 1.29+ | Deployment |

---

## 4. API Contracts

### 4.1 Dashboard Summary

```
GET /api/v1/insights360/dashboard?project_id={id}&period=7d

Response 200 OK:
{
  "project_id": "uuid",
  "period": "7d",
  "kpis": {
    "test_pass_rate": 94.2,
    "requirement_coverage": 93.3,
    "avg_testability_score": 0.82,
    "automation_pct": 78.5,
    "p95_latency_ms": 245,
    "open_vulnerabilities": 3,
    "security_posture": "green"
  },
  "trends": {
    "pass_rate_7d": [91.0, 92.5, 93.0, 93.8, 94.0, 94.1, 94.2],
    "coverage_7d": [88.0, 89.2, 90.5, 91.0, 92.0, 93.0, 93.3]
  },
  "alerts": [
    { "type": "regression", "message": "Pass rate dropped 2% on payment module", "severity": "warning" }
  ]
}
```

### 4.2 Generate AI Report

```
POST /api/v1/insights360/reports/generate
Content-Type: application/json

Request Body:
{
  "project_id": "uuid",
  "report_type": "release_readiness" | "weekly_digest" | "executive_summary",
  "period": "7d" | "14d" | "30d",
  "model_preference": "gemini" | "claude" | "auto",
  "max_words": 300
}

Response 202 Accepted:
{
  "report_id": "uuid",
  "status": "generating",
  "estimated_seconds": 15
}
```

### 4.3 Get Report

```
GET /api/v1/insights360/reports/{report_id}

Response 200 OK:
{
  "report_id": "uuid",
  "type": "release_readiness",
  "status": "completed",
  "narrative": "## Release Readiness Assessment\n\nThe QA cycle for Sprint 14 shows...",
  "confidence": 0.88,
  "data_sources": ["deepspeci", "casegeni", "auto-playpilot", "perf-xi", "secure-xi"],
  "model_used": "gemini-2.5-pro",
  "generated_at": "2026-04-10T08:00:00Z"
}
```

### 4.4 Export PDF / CSV

```
POST /api/v1/insights360/reports/{report_id}/export
Content-Type: application/json

Request Body:
{
  "format": "pdf" | "csv",
  "include_charts": true,
  "include_details": true
}

Response 200 OK:
{
  "download_url": "https://blob.../report-uuid.pdf",
  "expires_at": "2026-04-11T08:00:00Z"
}
```

### 4.5 Send Notification

```
POST /api/v1/insights360/notifications/send
Content-Type: application/json

Request Body:
{
  "report_id": "uuid",
  "channels": ["slack", "teams", "email"],
  "recipients": {
    "slack_channel": "#qa-reports",
    "teams_channel": "QA Team",
    "email_list": ["mgr@org.com", "tm@org.com"]
  }
}

Response 200 OK:
{
  "deliveries": [
    { "channel": "slack", "status": "sent" },
    { "channel": "teams", "status": "sent" },
    { "channel": "email", "status": "queued", "count": 2 }
  ]
}
```

### 4.6 Drill-Down by Solution

```
GET /api/v1/insights360/solutions/{solution_name}/metrics?period=7d

Response 200 OK:
{
  "solution": "auto-playpilot",
  "period": "7d",
  "metrics": {
    "total_runs": 42,
    "pass_rate": 94.2,
    "avg_duration_ms": 3200,
    "flaky_tests": 3,
    "top_failures": [
      { "test_case_id": "TC-CG-102", "fail_count": 4, "last_error": "timeout" }
    ]
  }
}
```

---

## 5. Data Model

```mermaid
erDiagram
    METRIC_EVENT {
        uuid id PK
        string source "deepspeci | casegeni | auto-playpilot | ..."
        string metric_type "pass_rate | coverage | sli | vulnerability"
        float value
        json metadata
        uuid project_id FK
        timestamp recorded_at
    }

    KPI_SNAPSHOT {
        uuid id PK
        uuid project_id FK
        float test_pass_rate
        float requirement_coverage
        float avg_testability
        float automation_pct
        float p95_latency_ms
        int open_vulnerabilities
        string security_posture "red | amber | green"
        timestamp snapshot_at
    }

    REPORT {
        uuid id PK
        uuid project_id FK
        string type "release_readiness | weekly_digest | executive_summary"
        text narrative
        float confidence
        json data_sources
        string model_used
        string status "generating | completed | failed"
        timestamp generated_at
    }

    NOTIFICATION {
        uuid id PK
        uuid report_id FK
        string channel "slack | teams | email"
        string status "queued | sent | failed"
        json recipients
        timestamp sent_at
    }

    ALERT {
        uuid id PK
        uuid project_id FK
        string type "regression | threshold_breach | anomaly"
        string severity "info | warning | critical"
        text message
        boolean acknowledged
        timestamp created_at
    }

    PROJECT {
        uuid id PK
        string name
        string tenant_id
    }

    PROJECT ||--o{ METRIC_EVENT : "receives"
    PROJECT ||--o{ KPI_SNAPSHOT : "tracked by"
    PROJECT ||--o{ REPORT : "has"
    PROJECT ||--o{ ALERT : "triggers"
    REPORT ||--o{ NOTIFICATION : "sent via"
```

### Key Tables

| Table | Rows (est.) | Indexes | Partitioning |
|---|---|---|---|
| `metric_event` | ~1 M per tenant/month | `(project_id, source, recorded_at)` | TimescaleDB hypertable by `recorded_at` |
| `kpi_snapshot` | ~365 per tenant/year | `(project_id, snapshot_at)` | — |
| `report` | ~500 per tenant | `(project_id, type)` | — |
| `alert` | ~2 K per tenant | `(project_id, acknowledged, severity)` | — |

---

## 6. Integration Patterns

### 6.1 Event Ingestion (Upstream)

All 6 sibling solutions publish events via **Redis pub/sub**:

| Source | Event Channel | Example Event |
|---|---|---|
| **DeepSpeci** | `qi:events:deepspeci` | `{ type: "requirement_validated", req_id, testability: 0.85 }` |
| **CaseGeni** | `qi:events:casegeni` | `{ type: "cases_generated", count: 15, coverage: 0.93 }` |
| **DataGeni** | `qi:events:datageni` | `{ type: "dataset_generated", rows: 10000, pii_clean: true }` |
| **Auto-PlayPilot** | `qi:events:playpilot` | `{ type: "run_complete", passed: 10, failed: 1 }` |
| **Perf-Xi** | `qi:events:perfxi` | `{ type: "sli_reported", p95: 245, slo_status: "pass" }` |
| **Secure-Xi** | `qi:events:securexi` | `{ type: "scan_complete", critical: 0, high: 2 }` |

### 6.2 Aggregation Pipeline

```mermaid
flowchart LR
    EVENTS[Redis Pub/Sub<br/>6 solution channels] --> INGEST[Event Ingester<br/>Validate · Normalise]
    INGEST --> TS[(TimescaleDB<br/>metric_event)]
    TS --> AGG[KPI Aggregator<br/>Scheduled every 5 min]
    AGG --> SNAP[(kpi_snapshot)]
    SNAP --> CACHE[Redis Cache<br/>Dashboard data]
    CACHE --> DASH[Dashboard UI]
    AGG --> ALERT_ENG[Alert Engine<br/>Threshold checks]
    ALERT_ENG --> NOTIFY[Notification Service]
```

### 6.3 Report Generation Flow

```
1. Scheduler / User triggers report generation
2. Insights360 API gathers KPI snapshots for the period
3. Report Agent receives structured metrics + historical trends
4. LLM (Gemini / Claude) generates narrative (≤ 300 words)
5. Confidence score attached; low confidence → human review flag
6. Report stored in PostgreSQL; notifications dispatched
```

---

## 7. Deployment Configuration

### 7.1 Kubernetes Resources

```yaml
# insights360-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: insights360-api
  namespace: zenseai-qi
spec:
  replicas: 3
  selector:
    matchLabels:
      app: insights360-api
  template:
    metadata:
      labels:
        app: insights360-api
    spec:
      containers:
        - name: insights360-api
          image: zenseai.azurecr.io/insights360-api:latest
          ports:
            - containerPort: 8003
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
                  name: insights360-secrets
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                secretKeyRef:
                  name: insights360-secrets
                  key: redis-url
            - name: LLM_API_KEY
              valueFrom:
                secretKeyRef:
                  name: insights360-secrets
                  key: llm-api-key
            - name: SLACK_WEBHOOK_URL
              valueFrom:
                secretKeyRef:
                  name: insights360-secrets
                  key: slack-webhook
          livenessProbe:
            httpGet:
              path: /health
              port: 8003
            initialDelaySeconds: 10
            periodSeconds: 30
---
# Event ingester (background worker)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: insights360-ingester
  namespace: zenseai-qi
spec:
  replicas: 2
  selector:
    matchLabels:
      app: insights360-ingester
  template:
    metadata:
      labels:
        app: insights360-ingester
    spec:
      containers:
        - name: ingester
          image: zenseai.azurecr.io/insights360-api:latest
          command: ["python", "-m", "app.ingester"]
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "1000m"
              memory: "1Gi"
```

### 7.2 Environment Variables

| Variable | Description | Example |
|---|---|---|
| `DATABASE_URL` | PostgreSQL + TimescaleDB | `postgresql://user:pass@host:5432/insights360` |
| `REDIS_URL` | Redis (pub/sub + cache) | `redis://host:6379/3` |
| `LLM_API_KEY` | Gemini / Claude key | `key-...` |
| `SLACK_WEBHOOK_URL` | Slack incoming webhook | `https://hooks.slack.com/...` |
| `TEAMS_WEBHOOK_URL` | Teams incoming webhook | `https://outlook.office.com/...` |
| `SMTP_HOST` | Email relay | `smtp.sendgrid.net` |
| `DAILY_DIGEST_CRON` | Digest schedule | `0 8 * * *` |
| `KPI_AGG_INTERVAL` | Aggregation frequency | `300` (seconds) |
| `LOG_LEVEL` | Log verbosity | `INFO` |

### 7.3 Scheduled Jobs

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-digest
  namespace: zenseai-qi
spec:
  schedule: "0 8 * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: digest
              image: zenseai.azurecr.io/insights360-api:latest
              command: ["python", "-m", "app.jobs.daily_digest"]
          restartPolicy: OnFailure
```

---

## 8. Folder Structure

```
insights360/
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── dashboard/          # KPI tiles + trends
│   │   │   ├── coverage/           # Heatmap view
│   │   │   ├── reports/            # AI narrative viewer
│   │   │   ├── drilldown/          # Per-solution detail
│   │   │   └── alerts/             # Alert management
│   │   ├── components/
│   │   │   ├── KPITile.tsx
│   │   │   ├── TrendChart.tsx      # Recharts wrapper
│   │   │   ├── CoverageHeatmap.tsx
│   │   │   ├── NarrativeViewer.tsx
│   │   │   └── AlertBanner.tsx
│   │   ├── hooks/
│   │   │   ├── useDashboard.ts
│   │   │   └── useReports.ts
│   │   └── lib/
│   │       ├── api-client.ts
│   │       └── types.ts
│   ├── package.json
│   └── tsconfig.json
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── ingester.py             # Redis pub/sub consumer
│   │   ├── routers/
│   │   │   ├── dashboard.py        # KPI + trends endpoints
│   │   │   ├── reports.py          # AI report generation
│   │   │   ├── export.py           # PDF / CSV export
│   │   │   ├── notifications.py    # Slack / Teams / Email
│   │   │   └── drilldown.py        # Per-solution metrics
│   │   ├── services/
│   │   │   ├── aggregator.py       # KPI computation engine
│   │   │   ├── agent_client.py     # Report Agent calls
│   │   │   ├── pdf_generator.py    # WeasyPrint integration
│   │   │   ├── notifier.py         # Multi-channel dispatch
│   │   │   └── alert_engine.py     # Threshold + anomaly checks
│   │   ├── models/
│   │   │   ├── metric_event.py
│   │   │   ├── kpi_snapshot.py
│   │   │   └── report.py
│   │   ├── schemas/
│   │   │   ├── dashboard.py
│   │   │   └── report.py
│   │   └── config.py
│   ├── jobs/
│   │   ├── daily_digest.py         # CronJob entrypoint
│   │   └── kpi_aggregator.py       # Periodic aggregation
│   ├── alembic/
│   ├── tests/
│   │   ├── test_aggregator.py
│   │   ├── test_narrative.py
│   │   └── test_notifier.py
│   ├── requirements.txt
│   └── Dockerfile
├── k8s/
│   ├── api-deployment.yaml
│   ├── ingester-deployment.yaml
│   ├── daily-digest-cronjob.yaml
│   ├── service.yaml
│   └── secrets.yaml
└── docker-compose.yaml
```

---

## 9. Security Considerations

| Concern | Mitigation |
|---|---|
| **Cross-tenant data** | Tenant ID filtering on all queries; row-level security in PostgreSQL |
| **Report confidentiality** | Reports stored per-tenant; PDF URLs with time-limited SAS tokens |
| **Notification credentials** | Slack/Teams webhooks in Key Vault; SMTP credentials encrypted |
| **Authentication** | Entra ID OAuth 2.0; read-only dashboard role available |
| **Authorisation** | Roles: Viewer, Reporter, Admin |
| **AI narrative accuracy** | Confidence score attached; < 0.70 triggers human review |
| **Rate limiting** | Report generation capped at 10/hour per tenant |

---

## 10. Performance Targets

| Metric | Target | Notes |
|---|---|---|
| Dashboard load (cold) | ≤ 3 s | Redis-cached KPI snapshots |
| Dashboard refresh (warm) | ≤ 500 ms | Cache hit |
| Event ingestion throughput | ≥ 5 K events/s | Redis pub/sub + batch insert |
| KPI aggregation (daily) | ≤ 30 s | TimescaleDB continuous aggregates |
| AI report generation | ≤ 15 s | Including LLM round-trip |
| PDF export | ≤ 10 s | WeasyPrint rendering |
| Notification dispatch | ≤ 2 s per channel | Async parallel send |
| Availability | 99.9 % | Multi-AZ, Redis sentinel |

---

## 11. Implementation Roadmap

| Phase | Timeline | Deliverables |
|---|---|---|
| **Phase 1 — Core** | Weeks 1–2 | Event ingestion, basic KPI computation, dashboard scaffold |
| **Phase 2 — Dashboards** | Weeks 3–4 | KPI tiles, trend charts, coverage heatmap, drill-down views |
| **Phase 3 — AI Reports** | Weeks 5–6 | Report Agent integration, narrative generation, confidence scoring |
| **Phase 4 — Notifications** | Weeks 7–8 | Slack, Teams, email dispatch; daily digest CronJob |
| **Phase 5 — Polish** | Weeks 9–10 | PDF/CSV export, alert engine, anomaly detection, performance tuning |

---

📄 [Full Requirements Spec →](../solutions/insights360.md) · 🤖 [Agent Design →](../agents/report-agent.md) · [← Platform Architecture](../architecture.md)
