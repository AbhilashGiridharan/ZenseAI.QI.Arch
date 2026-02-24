---
layout: default
title: "CaseGeni — Solution Architecture & Implementation"
description: "Detailed architecture, component design, API contracts, data model, and implementation guide for CaseGeni."
---

[← Back to Platform Architecture](../architecture.md) · [Requirements Spec](../solutions/casegeni.md) · [Agent Design](../agents/test-case-generator.md)
{: .fs-3 }

# CaseGeni — Solution Architecture & Implementation

---

## 1. Architecture Overview

<img src="../../assets/images/arch-casegeni.svg" alt="CaseGeni Architecture" width="100%" style="border-radius:6px; margin-bottom:1rem;" />

```mermaid
flowchart LR
    DS[DeepSpeci<br/>Validated Reqs] --> UI

    subgraph Users
        AE([Automation Engineer])
        TM([Technical Manager])
    end

    subgraph CaseGeni
        UI[CaseGeni UI<br/>Generate · Review · Approve]
        API[CaseGeni API]
    end

    subgraph Agent
        TCG[Test Case Generator<br/>Gherkin · BVA · Edge · Dedup]
    end

    subgraph Models["Model Providers"]
        GEM[Gemini]
        CLA[Claude]
    end

    DB[(PostgreSQL<br/>Cases · Matrix)]

    Users --> UI --> API --> TCG
    TCG --> GEM & CLA
    TCG -->|test cases| API
    API --> DB

    API -.->|cases| APP[Auto-PlayPilot]
    API -.->|data needs| DG[DataGeni]
    API -.->|coverage| I3[Insights360]
```

### Key Flows

- Validated requirements from **DeepSpeci** are the primary input
- The **Test Case Generator** produces Gherkin scenarios with BVA, EP, and negative/edge cases
- Deduplication (cosine similarity ≥ 0.92) prevents redundant tests
- Test cases flow to **Auto-PlayPilot** (automation), **DataGeni** (data needs), and **Insights360** (coverage)

---

## 2. Component Breakdown

```mermaid
flowchart TB
    subgraph FrontEnd["CaseGeni UI (React / Next.js)"]
        GEN[Generation Panel<br/>Req → Scenarios]
        REVIEW[Review & Edit<br/>Approve · Revise · Reject]
        MATRIX[Coverage Matrix<br/>Req ↔ Test Heatmap]
        FILTER[Filter & Search<br/>Tags · Priority · Status]
    end

    subgraph APILayer["CaseGeni API (FastAPI)"]
        GENSVC[Generation Service<br/>Agent orchestration]
        DEDUP[Dedup Service<br/>Cosine similarity ≥ 0.92]
        COV[Coverage Service<br/>Matrix computation]
        EXPS[Export Service<br/>Auto-PlayPilot · DataGeni push]
        TAG[Tagging Service<br/>BVA · EP · Negative · Edge]
    end

    subgraph AgentLayer["Test Case Generator Agent"]
        GHERKIN[Gherkin Generator]
        BVA[BVA / EP Analyser]
        EDGE[Edge-Case Synthesiser]
        DEDUPER[Semantic Deduplicator]
    end

    subgraph DataLayer["Data Stores"]
        PG[(PostgreSQL<br/>Test Cases · Coverage)]
        VEC[(Vector Store<br/>Embeddings for dedup)]
        REDIS[(Redis<br/>Generation jobs · Cache)]
    end

    FrontEnd --> APILayer
    APILayer --> AgentLayer
    APILayer --> DataLayer
```

| Component | Technology | Responsibility |
|---|---|---|
| **CaseGeni UI** | React 18 / Next.js 15, Tailwind CSS | Scenario generation, review, coverage matrix visualisation |
| **CaseGeni API** | Python 3.12, FastAPI, Uvicorn | Generation orchestration, dedup, coverage, export |
| **Test Case Generator** | Python 3.12, LangChain/LangGraph | Gherkin output, BVA/EP analysis, edge-case synthesis, dedup |
| **PostgreSQL** | PostgreSQL 16 | Test cases, coverage matrix, traceability links |
| **Vector Store** | pgvector extension | Embedding storage for semantic deduplication |
| **Redis** | Redis 7 | Async job queue, generation caching |

---

## 3. Tech Stack

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Frontend | React + Next.js | 18.x / 15.x | Scenario editor, coverage heatmap |
| Styling | Tailwind CSS | 3.x | Responsive UI components |
| State | Zustand | 4.x | Client-side state (scenarios, filters) |
| API Framework | FastAPI | 0.115+ | Async REST endpoints |
| Agent Framework | LangChain / LangGraph | 0.3+ | Multi-step generation chains |
| LLM Providers | Gemini, Claude | Latest | Gherkin generation, BVA analysis |
| Embeddings | OpenAI text-embedding-3-small | — | Semantic dedup vectors |
| Vector DB | pgvector | 0.7+ | Cosine similarity search |
| Database | PostgreSQL | 16 | Relational store |
| ORM | SQLAlchemy + Alembic | 2.x | Migrations & queries |
| Cache / Queue | Redis | 7.x | Job queue, rate-limit |
| Containerisation | Docker | 24+ | Packaging |
| Orchestration | Kubernetes (AKS) | 1.29+ | Deployment |

---

## 4. API Contracts

### 4.1 Generate Test Cases from Requirements

```
POST /api/v1/casegeni/generate
Content-Type: application/json

Request Body:
{
  "requirement_ids": ["REQ-042", "REQ-043"],
  "techniques": ["bva", "equivalence_partition", "negative", "edge"],
  "format": "gherkin",
  "model_preference": "gemini" | "claude" | "auto",
  "dedup_threshold": 0.92
}

Response 202 Accepted:
{
  "job_id": "uuid",
  "status": "generating",
  "estimated_seconds": 20
}
```

### 4.2 Get Generation Result

```
GET /api/v1/casegeni/jobs/{job_id}

Response 200 OK:
{
  "job_id": "uuid",
  "status": "completed",
  "test_cases": [
    {
      "tc_id": "TC-CG-101",
      "requirement_id": "REQ-042",
      "title": "Verify file upload with valid 10 MB file",
      "type": "positive",
      "technique": "bva",
      "priority": "high",
      "gherkin": "Feature: File Upload\n  Scenario: Upload valid file...",
      "data_needs": ["valid_file_10mb", "authenticated_user"],
      "tags": ["upload", "bva", "positive"]
    }
  ],
  "duplicates_removed": 3,
  "model_used": "gemini-2.5-pro",
  "generation_time_ms": 12400
}
```

### 4.3 Review Test Case

```
PATCH /api/v1/casegeni/test-cases/{tc_id}/review
Content-Type: application/json

Request Body:
{
  "action": "approve" | "revise" | "reject",
  "comment": "Edge case coverage looks good.",
  "revised_gherkin": "..."  // only if action == "revise"
}

Response 200 OK:
{
  "tc_id": "TC-CG-101",
  "status": "approved",
  "reviewed_by": "ae@org.com"
}
```

### 4.4 Coverage Matrix

```
GET /api/v1/casegeni/coverage?project_id={project_id}

Response 200 OK:
{
  "project_id": "uuid",
  "total_requirements": 45,
  "covered_requirements": 42,
  "coverage_pct": 93.3,
  "matrix": [
    { "req_id": "REQ-042", "test_cases": ["TC-CG-101", "TC-CG-102"], "coverage": 1.0 },
    { "req_id": "REQ-050", "test_cases": [], "coverage": 0.0 }
  ]
}
```

### 4.5 Export to Auto-PlayPilot / DataGeni

```
POST /api/v1/casegeni/export
Content-Type: application/json

Request Body:
{
  "test_case_ids": ["TC-CG-101", "TC-CG-102"],
  "targets": ["auto-playpilot", "datageni"]
}

Response 200 OK:
{
  "export_id": "uuid",
  "deliveries": [
    { "target": "auto-playpilot", "count": 2, "status": "delivered" },
    { "target": "datageni", "count": 2, "status": "delivered" }
  ]
}
```

---

## 5. Data Model

```mermaid
erDiagram
    TEST_CASE {
        uuid id PK
        string tc_id "TC-CG-101"
        uuid requirement_id FK
        string title
        text gherkin
        string type "positive | negative | edge | boundary"
        string technique "bva | ep | negative | edge | exploratory"
        string priority "critical | high | medium | low"
        string status "generated | approved | rejected | automated"
        json data_needs "['valid_file', 'auth_user']"
        json tags "['upload', 'bva']"
        vector embedding "1536-dim"
        uuid project_id FK
        timestamp created_at
        timestamp updated_at
    }

    COVERAGE_ENTRY {
        uuid id PK
        uuid requirement_id FK
        uuid test_case_id FK
        float coverage_score
        timestamp computed_at
    }

    GENERATION_JOB {
        uuid id PK
        json requirement_ids
        string status "queued | generating | completed | failed"
        string model_used
        int cases_generated
        int duplicates_removed
        int generation_time_ms
        timestamp created_at
    }

    REVIEW {
        uuid id PK
        uuid test_case_id FK
        string action "approve | revise | reject"
        string reviewer
        text comment
        timestamp created_at
    }

    PROJECT {
        uuid id PK
        string name
        string tenant_id
    }

    TEST_CASE ||--o{ COVERAGE_ENTRY : "covers"
    TEST_CASE ||--o{ REVIEW : "reviewed by"
    PROJECT ||--o{ TEST_CASE : "contains"
    GENERATION_JOB ||--o{ TEST_CASE : "produced"
```

### Key Tables

| Table | Rows (est.) | Indexes |
|---|---|---|
| `test_case` | ~50 K per tenant | `(project_id, status)`, `(requirement_id)`, IVFFlat on `embedding` |
| `coverage_entry` | ~100 K per tenant | `(requirement_id)`, `(test_case_id)` |
| `generation_job` | ~5 K per tenant | `(status, created_at)` |
| `review` | ~20 K per tenant | `(test_case_id, created_at)` |

---

## 6. Integration Patterns

### 6.1 Upstream Integrations

| Source | Protocol | Details |
|---|---|---|
| **DeepSpeci** | REST API (internal) | Receives validated requirement JSON; webhook on approval events |

### 6.2 Downstream Integrations

| Target | Protocol | Trigger | Payload |
|---|---|---|---|
| **Auto-PlayPilot** | REST API (internal) | Manual export or auto-on-approve | Approved test case JSON with Gherkin |
| **DataGeni** | REST API (internal) | Automatic on generation complete | Data needs array per test case |
| **Insights360** | Event bus (Redis pub/sub) | On generation + approval events | `{ tc_id, req_id, status, coverage }` |

### 6.3 Agent Integration

```
CaseGeni API  ──POST──▶  /api/agents/test-case-generator/generate
                          │
                          ├─ Pre-process: parse requirements, extract domains
                          ├─ Select technique chain (BVA, EP, negative, edge)
                          ├─ Route to model (Gemini for large context, Claude for nuance)
                          ├─ LLM call (temperature 0.2, structured Gherkin output)
                          ├─ Post-process: parse Gherkin, tag, prioritise
                          ├─ Dedup: embed generated cases → cosine similarity check
                          │   └─ Remove duplicates ≥ 0.92 similarity
                          └─ Return structured test cases → review queue
```

### 6.4 Deduplication Pipeline

```mermaid
flowchart LR
    NEW[New Test Case] --> EMB[Generate Embedding<br/>text-embedding-3-small]
    EMB --> SEARCH[Cosine Search<br/>pgvector]
    SEARCH --> CHECK{Similarity<br/>≥ 0.92?}
    CHECK -->|Yes| SKIP[Mark Duplicate<br/>Log & Skip]
    CHECK -->|No| STORE[Store Case<br/>+ Embedding]
```

---

## 7. Deployment Configuration

### 7.1 Kubernetes Resources

```yaml
# casegeni-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: casegeni-api
  namespace: zenseai-qi
spec:
  replicas: 3
  selector:
    matchLabels:
      app: casegeni-api
  template:
    metadata:
      labels:
        app: casegeni-api
    spec:
      containers:
        - name: casegeni-api
          image: zenseai.azurecr.io/casegeni-api:latest
          ports:
            - containerPort: 8001
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
                  name: casegeni-secrets
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                secretKeyRef:
                  name: casegeni-secrets
                  key: redis-url
            - name: LLM_API_KEY
              valueFrom:
                secretKeyRef:
                  name: casegeni-secrets
                  key: llm-api-key
            - name: EMBEDDING_API_KEY
              valueFrom:
                secretKeyRef:
                  name: casegeni-secrets
                  key: embedding-api-key
          livenessProbe:
            httpGet:
              path: /health
              port: 8001
            initialDelaySeconds: 10
            periodSeconds: 30
          readinessProbe:
            httpGet:
              path: /ready
              port: 8001
            initialDelaySeconds: 5
            periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: casegeni-api
  namespace: zenseai-qi
spec:
  selector:
    app: casegeni-api
  ports:
    - port: 80
      targetPort: 8001
  type: ClusterIP
```

### 7.2 Environment Variables

| Variable | Description | Example |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection (with pgvector) | `postgresql://user:pass@host:5432/casegeni` |
| `REDIS_URL` | Redis connection | `redis://host:6379/1` |
| `LLM_API_KEY` | Gemini / Claude API key | `key-...` |
| `EMBEDDING_API_KEY` | OpenAI embedding API key | `sk-...` |
| `DEDUP_THRESHOLD` | Cosine similarity dedup cutoff | `0.92` |
| `DEEPSPECI_API_URL` | DeepSpeci internal API | `http://deepspeci-api.zenseai-qi/api/v1` |
| `AUTO_PLAYPILOT_API_URL` | Auto-PlayPilot internal API | `http://autoplaypilot-api.zenseai-qi/api/v1` |
| `LOG_LEVEL` | Log verbosity | `INFO` |

### 7.3 Scaling Policy

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: casegeni-api-hpa
  namespace: zenseai-qi
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: casegeni-api
  minReplicas: 2
  maxReplicas: 8
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
casegeni/
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── generate/           # Generation panel
│   │   │   ├── review/             # Review & approval
│   │   │   ├── coverage/           # Coverage matrix heatmap
│   │   │   └── export/             # Export wizard
│   │   ├── components/
│   │   │   ├── GherkinEditor.tsx
│   │   │   ├── CoverageHeatmap.tsx
│   │   │   ├── TestCaseCard.tsx
│   │   │   └── TechniqueSelector.tsx
│   │   ├── hooks/
│   │   │   ├── useGeneration.ts
│   │   │   └── useCoverage.ts
│   │   └── lib/
│   │       ├── api-client.ts
│   │       └── types.ts
│   ├── package.json
│   └── tsconfig.json
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── routers/
│   │   │   ├── generation.py       # Generate endpoints
│   │   │   ├── test_cases.py       # CRUD + review
│   │   │   ├── coverage.py         # Matrix computation
│   │   │   └── export.py           # Downstream push
│   │   ├── services/
│   │   │   ├── agent_client.py     # Test Case Generator calls
│   │   │   ├── dedup.py            # Embedding + cosine dedup
│   │   │   ├── coverage.py         # Coverage matrix engine
│   │   │   └── export.py           # Auto-PlayPilot / DataGeni push
│   │   ├── models/
│   │   │   ├── test_case.py
│   │   │   └── generation_job.py
│   │   ├── schemas/
│   │   │   ├── generation.py
│   │   │   └── test_case.py
│   │   └── config.py
│   ├── alembic/
│   ├── tests/
│   │   ├── test_generation.py
│   │   ├── test_dedup.py
│   │   └── test_coverage.py
│   ├── requirements.txt
│   └── Dockerfile
├── agent/
│   ├── test_case_generator/
│   │   ├── chain.py                # LangChain generation chain
│   │   ├── prompts/
│   │   │   ├── gherkin.txt
│   │   │   ├── bva.txt
│   │   │   └── edge_case.txt
│   │   ├── dedup.py                # Embedding dedup logic
│   │   └── router.py               # Model selection
│   └── tests/
│       └── test_generator.py
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── hpa.yaml
│   └── secrets.yaml
└── docker-compose.yaml
```

---

## 9. Security Considerations

| Concern | Mitigation |
|---|---|
| **Test data leakage** | No production data in test case descriptions; Presidio scan on inputs |
| **Authentication** | Entra ID OAuth 2.0 with JWT; per-tenant isolation |
| **Authorisation** | Role-based: Generator, Reviewer, Approver, Admin |
| **Embedding storage** | Vectors stored in pgvector; tenant-scoped queries only |
| **LLM prompt injection** | Input sanitisation; structured output schema enforcement |
| **Data at rest** | AES-256 for PostgreSQL; encrypted blob backups |
| **Audit trail** | Every generation, review, export logged with user + timestamp |

---

## 10. Performance Targets

| Metric | Target | Notes |
|---|---|---|
| Test case generation (10 reqs) | ≤ 30 s | Async job with progress updates |
| Single requirement → scenarios | ≤ 8 s | Including LLM round-trip |
| Deduplication check per case | ≤ 200 ms | pgvector IVFFlat index |
| Coverage matrix (500 reqs) | ≤ 3 s | Pre-computed, Redis-cached |
| API throughput | ≥ 150 req/s | At 3 pod replicas |
| Availability | 99.9 % | Multi-AZ deployment |

---

## 11. Implementation Roadmap

| Phase | Timeline | Deliverables |
|---|---|---|
| **Phase 1 — Core** | Weeks 1–2 | Test case CRUD, basic Gherkin generation, UI scaffold |
| **Phase 2 — AI Techniques** | Weeks 3–4 | BVA/EP/negative/edge chains, multi-model routing |
| **Phase 3 — Dedup** | Weeks 5–6 | Embedding pipeline, pgvector dedup, threshold tuning |
| **Phase 4 — Integrations** | Weeks 7–8 | DeepSpeci ingest, Auto-PlayPilot + DataGeni export, Insights360 events |
| **Phase 5 — Polish** | Weeks 9–10 | Coverage matrix heatmap, review workflow, performance tuning |

---

📄 [Full Requirements Spec →](../solutions/casegeni.md) · 🤖 [Agent Design →](../agents/test-case-generator.md) · [← Platform Architecture](../architecture.md)
