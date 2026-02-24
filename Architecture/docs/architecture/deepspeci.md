---
layout: default
title: "DeepSpeci — Solution Architecture & Implementation"
description: "Detailed architecture, component design, API contracts, data model, and implementation guide for DeepSpeci."
---

[← Back to Platform Architecture](../architecture.md) · [Requirements Spec](../solutions/deepspeci.md) · [Agent Design](../agents/requirement-evaluator.md)
{: .fs-3 }

# DeepSpeci — Solution Architecture & Implementation

---

## 1. Architecture Overview

<img src="../../assets/images/arch-deepspeci.svg" alt="DeepSpeci Architecture" width="100%" style="border-radius:6px; margin-bottom:1rem;" />

```mermaid
flowchart LR
    subgraph Users
        NTU([Non-Technical User])
        TM([Technical Manager])
        SME([SME])
        DEV([Developer])
    end

    subgraph DeepSpeci
        UI[DeepSpeci UI<br/>Chat · Dashboard · Import]
        API[DeepSpeci API]
    end

    subgraph Agent
        RE[Requirement Evaluator<br/>Ambiguity · Testability · Gaps]
    end

    subgraph Models["Model Providers"]
        GEM[Gemini]
        CLA[Claude]
        OAI[OpenAI]
    end

    DB[(PostgreSQL<br/>Reqs · Versions)]

    Users --> UI --> API --> RE
    RE --> GEM & CLA & OAI
    RE -->|validated reqs| API
    API --> DB

    API -.->|export| CG[CaseGeni]
    API -.->|metrics| I3[Insights360]
```

### Key Flows

- Requirements are imported from JIRA, Confluence, DOCX, or entered via natural-language chat
- The **Requirement Evaluator** agent detects ambiguities, scores testability (0–1), and identifies gaps
- Validated requirements flow downstream to **CaseGeni** and coverage metrics to **Insights360**
- All changes are versioned with full audit trail in PostgreSQL

---

## 2. Component Breakdown

```mermaid
flowchart TB
    subgraph FrontEnd["DeepSpeci UI (React / Next.js)"]
        CHAT[Chat Interface<br/>Natural-language input]
        IMPORT[Import Wizard<br/>JIRA · Confluence · DOCX · MD]
        DASH[Validation Dashboard<br/>Scores · Gaps · Traceability]
        REVIEW[Review Workflow<br/>Approve · Revise · Reject]
    end

    subgraph APILayer["DeepSpeci API (FastAPI)"]
        INGR[Ingestion Service<br/>Parse · Chunk · Normalise]
        VALS[Validation Service<br/>Agent orchestration]
        TRCS[Traceability Service<br/>Req ↔ Test matrix]
        EXPS[Export Service<br/>JSON · CSV · CaseGeni push]
        VERS[Versioning Service<br/>Diff · Audit trail]
    end

    subgraph AgentLayer["Requirement Evaluator Agent"]
        AMB[Ambiguity Detector]
        TEST[Testability Scorer]
        GAP[Gap Analyser]
        STRUCT[Output Structurer]
    end

    subgraph DataLayer["Data Stores"]
        PG[(PostgreSQL<br/>Requirements · Versions · Audit)]
        REDIS[(Redis<br/>Session cache · Rate-limit)]
        BLOB[(Blob Storage<br/>Uploaded docs)]
    end

    FrontEnd --> APILayer
    APILayer --> AgentLayer
    APILayer --> DataLayer
```

| Component | Technology | Responsibility |
|---|---|---|
| **DeepSpeci UI** | React 18 / Next.js 15, Tailwind CSS | Chat input, import wizard, validation dashboard, review workflow |
| **DeepSpeci API** | Python 3.12, FastAPI, Uvicorn | Ingestion, validation orchestration, traceability, export, versioning |
| **Requirement Evaluator** | Python 3.12, LangChain/LangGraph | Ambiguity detection, testability scoring, gap analysis |
| **PostgreSQL** | PostgreSQL 16 | Requirements store, version history, audit log |
| **Redis** | Redis 7 | Session caching, rate-limiting, pub/sub events |
| **Blob Storage** | Azure Blob / S3 | Uploaded document storage (DOCX, PDF) |

---

## 3. Tech Stack

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Frontend | React + Next.js | 18.x / 15.x | SPA with SSR, chat UI, dashboards |
| Styling | Tailwind CSS | 3.x | Utility-first responsive styling |
| State | Zustand | 4.x | Client-side state management |
| API Framework | FastAPI | 0.115+ | Async REST + WebSocket endpoints |
| Agent Framework | LangChain / LangGraph | 0.3+ | LLM orchestration, chain composition |
| LLM Providers | Gemini, Claude, OpenAI | Latest | Multi-model requirement analysis |
| PII Protection | Presidio | 2.x | PII detection & redaction pre/post LLM |
| Database | PostgreSQL | 16 | Relational data store |
| ORM | SQLAlchemy + Alembic | 2.x | Schema management & migrations |
| Cache | Redis | 7.x | Session cache, pub/sub |
| Object Storage | Azure Blob / S3 | — | Document uploads |
| Containerisation | Docker | 24+ | Service packaging |
| Orchestration | Kubernetes (AKS) | 1.29+ | Deployment & scaling |

---

## 4. API Contracts

### 4.1 Requirement Ingestion

```
POST /api/v1/deepspeci/requirements/import
Content-Type: multipart/form-data

Parameters:
  source: "jira" | "confluence" | "docx" | "markdown" | "chat"
  project_id: string         (JIRA project key, optional)
  file: binary               (DOCX/MD file upload, optional)
  text: string               (natural-language input, optional)

Response 202 Accepted:
{
  "job_id": "uuid",
  "status": "processing",
  "estimated_seconds": 15
}
```

### 4.2 Validate Requirements

```
POST /api/v1/deepspeci/requirements/{req_id}/validate
Content-Type: application/json

Request Body:
{
  "model_preference": "gemini" | "claude" | "openai" | "auto",
  "checks": ["ambiguity", "testability", "gaps", "all"]
}

Response 200 OK:
{
  "req_id": "REQ-042",
  "ambiguity": {
    "score": 0.72,
    "flags": [
      { "term": "large", "suggestion": "Define as > 10 MB", "confidence": 0.91 }
    ]
  },
  "testability": {
    "score": 0.85,
    "reason": "Clear acceptance criteria; measurable outcome"
  },
  "gaps": [
    { "type": "missing_precondition", "severity": "medium", "suggestion": "Add user role constraint" }
  ],
  "model_used": "claude-sonnet-4-20250514",
  "latency_ms": 2340
}
```

### 4.3 Export to CaseGeni

```
POST /api/v1/deepspeci/requirements/export
Content-Type: application/json

Request Body:
{
  "requirement_ids": ["REQ-042", "REQ-043", "REQ-044"],
  "target": "casegeni",
  "format": "structured_json"
}

Response 200 OK:
{
  "export_id": "uuid",
  "target": "casegeni",
  "count": 3,
  "status": "delivered"
}
```

### 4.4 Get Requirement with Traceability

```
GET /api/v1/deepspeci/requirements/{req_id}?include=traceability,versions

Response 200 OK:
{
  "req_id": "REQ-042",
  "title": "User file upload",
  "description": "Users shall be able to upload files up to 50 MB.",
  "testability_score": 0.85,
  "ambiguity_score": 0.72,
  "status": "validated",
  "traceability": {
    "test_cases": ["TC-CG-101", "TC-CG-102"],
    "coverage": 0.67
  },
  "versions": [
    { "version": 2, "changed_by": "user@org.com", "timestamp": "2026-04-10T09:30:00Z" }
  ]
}
```

### 4.5 Review Workflow

```
PATCH /api/v1/deepspeci/requirements/{req_id}/review
Content-Type: application/json

Request Body:
{
  "action": "approve" | "revise" | "reject",
  "comment": "Accepted with minor edits.",
  "revised_text": "..."  // only if action == "revise"
}

Response 200 OK:
{
  "req_id": "REQ-042",
  "status": "approved",
  "reviewed_by": "tm@org.com",
  "reviewed_at": "2026-04-10T10:15:00Z"
}
```

---

## 5. Data Model

```mermaid
erDiagram
    REQUIREMENT {
        uuid id PK
        string title
        text description
        string source "jira | confluence | docx | chat"
        string source_ref "JIRA-123"
        float testability_score
        float ambiguity_score
        string status "draft | validating | validated | approved | rejected"
        uuid project_id FK
        timestamp created_at
        timestamp updated_at
    }

    REQUIREMENT_VERSION {
        uuid id PK
        uuid requirement_id FK
        int version_number
        text previous_text
        text new_text
        string changed_by
        string change_reason
        timestamp created_at
    }

    AMBIGUITY_FLAG {
        uuid id PK
        uuid requirement_id FK
        string term
        string suggestion
        float confidence
        string resolution "pending | accepted | dismissed"
    }

    GAP_FINDING {
        uuid id PK
        uuid requirement_id FK
        string gap_type "missing_precondition | incomplete_criteria | undefined_term"
        string severity "low | medium | high"
        string suggestion
        string resolution "pending | resolved | deferred"
    }

    REVIEW {
        uuid id PK
        uuid requirement_id FK
        string action "approve | revise | reject"
        string reviewer
        text comment
        timestamp created_at
    }

    PROJECT {
        uuid id PK
        string name
        string tenant_id
        timestamp created_at
    }

    REQUIREMENT ||--o{ REQUIREMENT_VERSION : "has versions"
    REQUIREMENT ||--o{ AMBIGUITY_FLAG : "has flags"
    REQUIREMENT ||--o{ GAP_FINDING : "has gaps"
    REQUIREMENT ||--o{ REVIEW : "has reviews"
    PROJECT ||--o{ REQUIREMENT : "contains"
```

### Key Tables

| Table | Rows (est.) | Indexes |
|---|---|---|
| `requirement` | ~10 K per tenant | `(project_id, status)`, `(source_ref)` |
| `requirement_version` | ~30 K per tenant | `(requirement_id, version_number)` |
| `ambiguity_flag` | ~50 K per tenant | `(requirement_id, resolution)` |
| `gap_finding` | ~20 K per tenant | `(requirement_id, severity)` |
| `review` | ~15 K per tenant | `(requirement_id, created_at)` |

---

## 6. Integration Patterns

### 6.1 Upstream Integrations

| Source | Protocol | Details |
|---|---|---|
| **JIRA** | REST API v3 | OAuth 2.0; import user stories, epics; webhook for real-time sync |
| **Confluence** | REST API v2 | Pull BRDs/PRDs by space + page ID; Markdown extraction |
| **DOCX uploads** | HTTP multipart | python-docx parsing → structured sections |
| **Chat input** | WebSocket | Real-time NL processing via UI chat interface |

### 6.2 Downstream Integrations

| Target | Protocol | Trigger | Payload |
|---|---|---|---|
| **CaseGeni** | REST API (internal) | Manual export or auto-on-approve | Validated requirement JSON array |
| **Insights360** | Event bus (Redis pub/sub) | On every validation complete | `{ req_id, testability, ambiguity, status }` |

### 6.3 Agent Integration

```
DeepSpeci API  ──POST──▶  /api/agents/requirement-evaluator/validate
                           │
                           ├─ Pre-process: sanitise, PII redact (Presidio)
                           ├─ Chunk if input > 50 K tokens
                           ├─ Route to model (Gemini / Claude / OpenAI)
                           ├─ LLM call (temperature 0.1, max 8 K completion tokens)
                           ├─ Post-process: parse structured output, score
                           ├─ Hallucination check: every claim → source req ID
                           └─ Return structured result → human review queue
```

---

## 7. Deployment Configuration

### 7.1 Kubernetes Resources

```yaml
# deepspeci-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deepspeci-api
  namespace: zenseai-qi
spec:
  replicas: 3
  selector:
    matchLabels:
      app: deepspeci-api
  template:
    metadata:
      labels:
        app: deepspeci-api
    spec:
      containers:
        - name: deepspeci-api
          image: zenseai.azurecr.io/deepspeci-api:latest
          ports:
            - containerPort: 8000
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
                  name: deepspeci-secrets
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                secretKeyRef:
                  name: deepspeci-secrets
                  key: redis-url
            - name: LLM_API_KEY
              valueFrom:
                secretKeyRef:
                  name: deepspeci-secrets
                  key: llm-api-key
          livenessProbe:
            httpGet:
              path: /health
              port: 8000
            initialDelaySeconds: 10
            periodSeconds: 30
          readinessProbe:
            httpGet:
              path: /ready
              port: 8000
            initialDelaySeconds: 5
            periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: deepspeci-api
  namespace: zenseai-qi
spec:
  selector:
    app: deepspeci-api
  ports:
    - port: 80
      targetPort: 8000
  type: ClusterIP
```

### 7.2 Environment Variables

| Variable | Description | Example |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://user:pass@host:5432/deepspeci` |
| `REDIS_URL` | Redis connection string | `redis://host:6379/0` |
| `LLM_API_KEY` | Model provider API key | `sk-...` |
| `LLM_DEFAULT_MODEL` | Default model for validation | `gemini-2.5-pro` |
| `JIRA_BASE_URL` | JIRA instance URL | `https://org.atlassian.net` |
| `JIRA_API_TOKEN` | JIRA API authentication token | `token-...` |
| `PII_REDACTION_ENABLED` | Toggle Presidio PII scanning | `true` |
| `LOG_LEVEL` | Application log level | `INFO` |

### 7.3 Scaling Policy

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: deepspeci-api-hpa
  namespace: zenseai-qi
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: deepspeci-api
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## 8. Folder Structure

```
deepspeci/
├── frontend/
│   ├── src/
│   │   ├── app/                    # Next.js app router
│   │   │   ├── chat/               # NL chat interface
│   │   │   ├── import/             # Import wizard pages
│   │   │   ├── dashboard/          # Validation dashboard
│   │   │   └── review/             # Approval workflow
│   │   ├── components/
│   │   │   ├── RequirementCard.tsx
│   │   │   ├── AmbiguityBadge.tsx
│   │   │   ├── TestabilityGauge.tsx
│   │   │   └── TraceabilityMatrix.tsx
│   │   ├── hooks/
│   │   │   ├── useRequirements.ts
│   │   │   └── useValidation.ts
│   │   └── lib/
│   │       ├── api-client.ts
│   │       └── types.ts
│   ├── package.json
│   └── tsconfig.json
├── backend/
│   ├── app/
│   │   ├── main.py                 # FastAPI entrypoint
│   │   ├── routers/
│   │   │   ├── requirements.py     # CRUD + import/export
│   │   │   ├── validation.py       # Agent orchestration
│   │   │   └── review.py           # Approval workflow
│   │   ├── services/
│   │   │   ├── ingestion.py        # JIRA, Confluence, DOCX parsers
│   │   │   ├── agent_client.py     # Requirement Evaluator API calls
│   │   │   ├── traceability.py     # Req ↔ Test mapping
│   │   │   └── export.py           # CaseGeni push
│   │   ├── models/
│   │   │   ├── requirement.py      # SQLAlchemy models
│   │   │   └── review.py
│   │   ├── schemas/
│   │   │   ├── requirement.py      # Pydantic request/response
│   │   │   └── validation.py
│   │   └── config.py               # Settings & env vars
│   ├── alembic/                    # DB migrations
│   ├── tests/
│   │   ├── test_ingestion.py
│   │   ├── test_validation.py
│   │   └── test_export.py
│   ├── requirements.txt
│   └── Dockerfile
├── agent/
│   ├── requirement_evaluator/
│   │   ├── chain.py                # LangChain/LangGraph chain
│   │   ├── prompts/
│   │   │   ├── ambiguity.txt
│   │   │   ├── testability.txt
│   │   │   └── gap_analysis.txt
│   │   ├── tools.py                # Presidio, schema validator
│   │   └── router.py               # Model selection logic
│   └── tests/
│       └── test_evaluator.py
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── hpa.yaml
│   └── secrets.yaml
└── docker-compose.yaml             # Local dev environment
```

---

## 9. Security Considerations

| Concern | Mitigation |
|---|---|
| **PII in requirements** | Presidio scans all inputs before LLM call; PII redacted → restored post-processing |
| **Authentication** | Entra ID (Azure AD) OAuth 2.0 with JWT; RBAC per tenant |
| **Authorisation** | Role-based: Viewer, Editor, Approver, Admin |
| **Data at rest** | AES-256 encryption for PostgreSQL, Blob Storage |
| **Data in transit** | TLS 1.3 for all API calls; mTLS between internal services |
| **LLM API keys** | Stored in Azure Key Vault; rotated every 90 days |
| **Audit logging** | Every action (create, validate, approve) logged with user, timestamp, IP |
| **Input sanitisation** | HTML/script injection prevention; max input size 10 MB |

---

## 10. Performance Targets

| Metric | Target | Notes |
|---|---|---|
| Requirement import (JIRA) | ≤ 30 s for 500 stories | Async job with progress callback |
| Single requirement validation (p95) | ≤ 5 s | Including LLM round-trip |
| Batch validation (100 reqs) | ≤ 90 s | Parallelised agent calls |
| Dashboard load | ≤ 2 s | Redis-cached aggregations |
| Traceability matrix generation | ≤ 10 s | Pre-computed on validation complete |
| API throughput | ≥ 200 req/s | At 3 pod replicas |
| Availability | 99.9 % | Multi-AZ deployment |

---

## 11. Implementation Roadmap

| Phase | Timeline | Deliverables |
|---|---|---|
| **Phase 1 — Core** | Weeks 1–2 | Requirement CRUD, DOCX/MD import, basic validation API |
| **Phase 2 — AI Integration** | Weeks 3–4 | Requirement Evaluator agent, ambiguity detection, testability scoring |
| **Phase 3 — Integrations** | Weeks 5–6 | JIRA/Confluence import, CaseGeni export, Insights360 event publishing |
| **Phase 4 — Review Workflow** | Weeks 7–8 | Approve/revise/reject workflow, version history, audit trail |
| **Phase 5 — Polish** | Weeks 9–10 | Dashboard refinement, traceability matrix, performance tuning, E2E tests |

---

📄 [Full Requirements Spec →](../solutions/deepspeci.md) · 🤖 [Agent Design →](../agents/requirement-evaluator.md) · [← Platform Architecture](../architecture.md)
