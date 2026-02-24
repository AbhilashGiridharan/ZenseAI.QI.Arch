---
layout: default
title: "Auto-PlayPilot — Solution Architecture & Implementation"
description: "Detailed architecture, component design, API contracts, data model, and implementation guide for Auto-PlayPilot."
---

[← Back to Platform Architecture](../architecture.md) · [Requirements Spec](../solutions/auto-playpilot.md) · [Agent Design](../agents/playwright-mcp.md)
{: .fs-3 }

# Auto-PlayPilot — Solution Architecture & Implementation

---

## 1. Architecture Overview

<img src="../../assets/images/arch-auto-playpilot.svg" alt="Auto-PlayPilot Architecture" width="100%" style="border-radius:6px; margin-bottom:1rem;" />

```mermaid
flowchart LR
    CG[CaseGeni<br/>Test Cases] --> UI
    DG[DataGeni<br/>Fixtures] --> UI

    subgraph Users
        AE([Automation Engineer])
        TM([Technical Manager])
    end

    subgraph AutoPlayPilot["Auto-PlayPilot"]
        UI[PlayPilot UI<br/>Generate · Review · Execute]
        API[PlayPilot API]
    end

    subgraph Agent
        PMCP[Playwright MCP<br/>Codegen · POM · Troubleshoot]
    end

    subgraph Toolchain
        ES[ESLint]
        TS[TypeScript]
        PW[PW Runner]
    end

    subgraph Models["Model Providers"]
        OAI[OpenAI]
        CLA[Claude]
    end

    VCS[(Git Repo<br/>Scripts · Screenshots)]

    Users --> UI --> API --> PMCP
    PMCP --> OAI & CLA
    PMCP --> ES & TS & PW
    PMCP -->|.spec.ts + POM| API
    API --> VCS

    API -.->|results| I3[Insights360]
```

### Key Flows

- Test cases from **CaseGeni** + fixtures from **DataGeni** are combined
- The **Playwright MCP** agent generates `.spec.ts` scripts with Page Object Models
- Scripts pass ESLint + TypeScript compilation gates before delivery
- AI troubleshooting diagnoses failing selectors and suggests code patches
- Execution results + screenshots flow to **Insights360**; scripts commit to VCS

---

## 2. Component Breakdown

```mermaid
flowchart TB
    subgraph FrontEnd["Auto-PlayPilot UI (React / Next.js)"]
        CODEGEN[Codegen Panel<br/>Test case → Script]
        EDITOR[Script Editor<br/>Monaco + Playwright snippets]
        RUNNER[Execution Console<br/>Run · Trace · Screenshots]
        TROUBLE[Troubleshoot Panel<br/>AI fix suggestions]
    end

    subgraph APILayer["PlayPilot API (Node.js / Express)"]
        GENSVC[Code Generation Service<br/>Agent orchestration]
        LINT[Lint & Compile Gate<br/>ESLint + tsc]
        EXEC[Execution Service<br/>PW Runner wrapper]
        VCS_SVC[VCS Service<br/>Git commit + PR]
        TROUBLESVC[Troubleshoot Service<br/>Failure analysis]
    end

    subgraph AgentLayer["Playwright MCP Agent"]
        CODEGEN_A[Script Generator<br/>.spec.ts + POM]
        POM_A[POM Builder<br/>Page Object patterns]
        SELECTOR[Selector Resolver<br/>AI-resilient locators]
        FIX[AI Troubleshooter<br/>Failure → patch]
    end

    subgraph DataLayer["Data Stores"]
        GIT[(Git Repository<br/>Scripts · POMs · Config)]
        PG[(PostgreSQL<br/>Runs · Results · Traces)]
        S3[(Blob Storage<br/>Screenshots · Videos · Traces)]
    end

    FrontEnd --> APILayer
    APILayer --> AgentLayer
    APILayer --> DataLayer
```

| Component | Technology | Responsibility |
|---|---|---|
| **Auto-PlayPilot UI** | React 18 / Next.js 15, Monaco Editor | Script editor, execution console, troubleshoot panel |
| **PlayPilot API** | Node.js 22, Express / Fastify | Code gen orchestration, lint gate, execution, VCS |
| **Playwright MCP** | TypeScript, LangChain.js | Script generation, POM, selector resolution, troubleshooting |
| **Playwright Runner** | Playwright Test 1.48+ | Headless/headed test execution, tracing |
| **Git Repository** | GitHub / GitLab | Script version control, PR creation |
| **PostgreSQL** | PostgreSQL 16 | Execution results, run history |
| **Blob Storage** | Azure Blob / S3 | Screenshots, videos, trace files |

---

## 3. Tech Stack

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Frontend | React + Next.js | 18.x / 15.x | Script IDE, execution dashboard |
| Code Editor | Monaco Editor | 0.50+ | Syntax highlighting, IntelliSense for Playwright |
| API Framework | Node.js + Fastify | 22.x / 5.x | High-perf API; closer to Playwright ecosystem |
| Agent Framework | LangChain.js | 0.3+ | LLM orchestration for code generation |
| LLM Providers | OpenAI, Claude | Latest | Code generation, troubleshooting |
| Test Framework | Playwright Test | 1.48+ | E2E test execution, tracing |
| Linting | ESLint + Prettier | 9.x / 3.x | Code quality gates |
| TypeScript | TypeScript | 5.6+ | Type-safe script generation |
| VCS | GitHub API / GitLab API | v4 | Commit, branch, PR automation |
| Database | PostgreSQL | 16 | Run results, metadata |
| Object Storage | Azure Blob / S3 | — | Screenshots, traces, videos |
| CI Integration | GitHub Actions / Azure Pipelines | — | Triggered test execution |
| Containerisation | Docker | 24+ | Packaging |
| Orchestration | Kubernetes (AKS) | 1.29+ | Deployment |

---

## 4. API Contracts

### 4.1 Generate Playwright Script

```
POST /api/v1/playpilot/generate
Content-Type: application/json

Request Body:
{
  "test_case_ids": ["TC-CG-101", "TC-CG-102"],
  "base_url": "https://staging.app.com",
  "fixtures": {
    "users": "dataset-uuid",
    "orders": "dataset-uuid"
  },
  "options": {
    "page_object_model": true,
    "selector_strategy": "data-testid" | "role" | "text" | "auto",
    "auth_pattern": "cookie" | "bearer" | "none",
    "parallel": false
  },
  "model_preference": "openai" | "claude" | "auto"
}

Response 202 Accepted:
{
  "job_id": "uuid",
  "status": "generating",
  "estimated_seconds": 25
}
```

### 4.2 Get Generated Script

```
GET /api/v1/playpilot/jobs/{job_id}

Response 200 OK:
{
  "job_id": "uuid",
  "status": "completed",
  "scripts": [
    {
      "script_id": "uuid",
      "test_case_id": "TC-CG-101",
      "filename": "file-upload.spec.ts",
      "code": "import { test, expect } from '@playwright/test';\n...",
      "page_objects": [
        { "filename": "pages/UploadPage.ts", "code": "..." }
      ],
      "lint_passed": true,
      "compile_passed": true
    }
  ],
  "model_used": "gpt-4.1",
  "generation_time_ms": 18500
}
```

### 4.3 Execute Test

```
POST /api/v1/playpilot/execute
Content-Type: application/json

Request Body:
{
  "script_ids": ["uuid-1", "uuid-2"],
  "environment": "staging" | "qa" | "local",
  "browser": "chromium" | "firefox" | "webkit" | "all",
  "headed": false,
  "trace": true,
  "retries": 2,
  "workers": 4
}

Response 202 Accepted:
{
  "run_id": "uuid",
  "status": "running",
  "estimated_seconds": 60
}
```

### 4.4 Get Execution Results

```
GET /api/v1/playpilot/runs/{run_id}

Response 200 OK:
{
  "run_id": "uuid",
  "status": "completed",
  "summary": {
    "total": 12,
    "passed": 10,
    "failed": 1,
    "skipped": 1,
    "duration_ms": 45000
  },
  "results": [
    {
      "script_id": "uuid",
      "test_case_id": "TC-CG-101",
      "status": "passed",
      "duration_ms": 3200,
      "screenshots": ["https://blob.../ss-1.png"],
      "trace_url": "https://blob.../trace.zip"
    },
    {
      "script_id": "uuid",
      "test_case_id": "TC-CG-102",
      "status": "failed",
      "error": "locator('.submit-btn') not found",
      "screenshots": ["https://blob.../ss-fail.png"],
      "troubleshoot_available": true
    }
  ]
}
```

### 4.5 AI Troubleshoot Failure

```
POST /api/v1/playpilot/troubleshoot
Content-Type: application/json

Request Body:
{
  "run_id": "uuid",
  "script_id": "uuid",
  "error_message": "locator('.submit-btn') not found",
  "screenshot_url": "https://blob.../ss-fail.png",
  "page_html_snapshot": "..."
}

Response 200 OK:
{
  "diagnosis": "Button changed from class 'submit-btn' to data-testid 'submit-form'",
  "suggested_fix": {
    "old_selector": ".submit-btn",
    "new_selector": "[data-testid='submit-form']",
    "confidence": 0.94
  },
  "patch": "--- a/file-upload.spec.ts\n+++ b/file-upload.spec.ts\n...",
  "model_used": "claude-sonnet-4-20250514"
}
```

### 4.6 Commit to VCS

```
POST /api/v1/playpilot/vcs/commit
Content-Type: application/json

Request Body:
{
  "script_ids": ["uuid-1"],
  "repo": "org/e2e-tests",
  "branch": "auto/TC-CG-101",
  "commit_message": "feat(e2e): add file upload test TC-CG-101",
  "create_pr": true
}

Response 201 Created:
{
  "commit_sha": "abc123",
  "branch": "auto/TC-CG-101",
  "pr_url": "https://github.com/org/e2e-tests/pull/42"
}
```

---

## 5. Data Model

```mermaid
erDiagram
    SCRIPT {
        uuid id PK
        string test_case_id FK
        string filename "file-upload.spec.ts"
        text code
        boolean lint_passed
        boolean compile_passed
        string status "generated | reviewed | committed"
        string model_used
        uuid project_id FK
        timestamp created_at
    }

    PAGE_OBJECT {
        uuid id PK
        uuid script_id FK
        string filename "pages/UploadPage.ts"
        text code
        json selectors "locator strategies"
    }

    EXECUTION_RUN {
        uuid id PK
        json script_ids
        string environment
        string browser
        int total
        int passed
        int failed
        int skipped
        int duration_ms
        timestamp started_at
        timestamp completed_at
    }

    TEST_RESULT {
        uuid id PK
        uuid run_id FK
        uuid script_id FK
        string status "passed | failed | skipped | flaky"
        int duration_ms
        text error_message
        json screenshots "blob URLs"
        string trace_url
        timestamp created_at
    }

    TROUBLESHOOT_LOG {
        uuid id PK
        uuid test_result_id FK
        text diagnosis
        json suggested_fix
        text patch
        float confidence
        string model_used
        timestamp created_at
    }

    VCS_COMMIT {
        uuid id PK
        uuid script_id FK
        string repo
        string branch
        string commit_sha
        string pr_url
        timestamp created_at
    }

    SCRIPT ||--o{ PAGE_OBJECT : "has POMs"
    SCRIPT ||--o{ TEST_RESULT : "executed as"
    SCRIPT ||--o{ VCS_COMMIT : "committed via"
    EXECUTION_RUN ||--o{ TEST_RESULT : "contains"
    TEST_RESULT ||--o| TROUBLESHOOT_LOG : "diagnosed by"
```

---

## 6. Integration Patterns

### 6.1 Upstream Integrations

| Source | Protocol | Details |
|---|---|---|
| **CaseGeni** | REST API (internal) | Approved Gherkin test cases → code generation input |
| **DataGeni** | REST API (internal) | JSON fixtures loaded into test setup (`beforeAll`) |

### 6.2 Downstream Integrations

| Target | Protocol | Trigger | Payload |
|---|---|---|---|
| **Insights360** | Event bus (Redis pub/sub) | On run complete | `{ run_id, passed, failed, duration }` |
| **Git Repository** | GitHub / GitLab REST API | Manual or auto-commit | `.spec.ts` + POM files + fixtures |
| **CI Pipeline** | GitHub Actions / Azure Pipelines | Webhook on PR merge | Triggered E2E test suite |

### 6.3 Code Generation Pipeline

```mermaid
flowchart TB
    TC[Test Case<br/>Gherkin from CaseGeni] --> PARSE[Parse Gherkin Steps]
    FIX[Fixtures<br/>from DataGeni] --> SETUP[Generate beforeAll Setup]
    PARSE --> GEN[Playwright MCP Agent<br/>Generate .spec.ts]
    SETUP --> GEN
    GEN --> POM[Extract Page Objects<br/>Build POM classes]
    POM --> LINT[ESLint Check]
    LINT --> TSC[TypeScript Compile]
    TSC --> PASS{Pass?}
    PASS -->|Yes| DELIVER[Deliver to Review Queue]
    PASS -->|No| RETRY[AI Self-Heal<br/>Fix lint/compile errors]
    RETRY --> LINT
```

---

## 7. Deployment Configuration

### 7.1 Kubernetes Resources

```yaml
# auto-playpilot-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: playpilot-api
  namespace: zenseai-qi
spec:
  replicas: 3
  selector:
    matchLabels:
      app: playpilot-api
  template:
    metadata:
      labels:
        app: playpilot-api
    spec:
      containers:
        - name: playpilot-api
          image: zenseai.azurecr.io/playpilot-api:latest
          ports:
            - containerPort: 3000
          resources:
            requests:
              cpu: "500m"
              memory: "1Gi"
            limits:
              cpu: "2000m"
              memory: "4Gi"
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: playpilot-secrets
                  key: database-url
            - name: OPENAI_API_KEY
              valueFrom:
                secretKeyRef:
                  name: playpilot-secrets
                  key: openai-api-key
            - name: GITHUB_TOKEN
              valueFrom:
                secretKeyRef:
                  name: playpilot-secrets
                  key: github-token
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 30
---
# Playwright runner pods (ephemeral, scaled per demand)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: playpilot-runner
  namespace: zenseai-qi
spec:
  replicas: 2
  selector:
    matchLabels:
      app: playpilot-runner
  template:
    metadata:
      labels:
        app: playpilot-runner
    spec:
      containers:
        - name: runner
          image: mcr.microsoft.com/playwright:v1.48.0-noble
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
| `DATABASE_URL` | PostgreSQL connection | `postgresql://user:pass@host:5432/playpilot` |
| `OPENAI_API_KEY` | OpenAI API key | `sk-...` |
| `ANTHROPIC_API_KEY` | Claude API key | `sk-ant-...` |
| `GITHUB_TOKEN` | GitHub PAT for VCS | `ghp_...` |
| `GITHUB_REPO` | Default target repo | `org/e2e-tests` |
| `BLOB_STORAGE_URL` | Screenshot/trace storage | `https://storage.blob.core.windows.net/...` |
| `BASE_URL` | Default SUT URL | `https://staging.app.com` |
| `PW_BROWSERS` | Installed browsers | `chromium,firefox` |
| `LOG_LEVEL` | Log verbosity | `INFO` |

### 7.3 Scaling Policy

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: playpilot-runner-hpa
  namespace: zenseai-qi
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: playpilot-runner
  minReplicas: 1
  maxReplicas: 15
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
```

---

## 8. Folder Structure

```
auto-playpilot/
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── generate/           # Code generation panel
│   │   │   ├── editor/             # Monaco script editor
│   │   │   ├── execute/            # Execution console
│   │   │   └── troubleshoot/       # AI fix panel
│   │   ├── components/
│   │   │   ├── ScriptEditor.tsx    # Monaco wrapper
│   │   │   ├── ExecutionLog.tsx
│   │   │   ├── ScreenshotViewer.tsx
│   │   │   └── TroubleshootCard.tsx
│   │   ├── hooks/
│   │   │   ├── useCodegen.ts
│   │   │   └── useExecution.ts
│   │   └── lib/
│   │       ├── api-client.ts
│   │       └── types.ts
│   ├── package.json
│   └── tsconfig.json
├── api/
│   ├── src/
│   │   ├── index.ts                # Fastify entrypoint
│   │   ├── routes/
│   │   │   ├── generate.ts         # Code gen endpoints
│   │   │   ├── execute.ts          # Run endpoints
│   │   │   ├── troubleshoot.ts     # AI fix endpoints
│   │   │   └── vcs.ts              # Git commit/PR
│   │   ├── services/
│   │   │   ├── agent-client.ts     # Playwright MCP calls
│   │   │   ├── lint-gate.ts        # ESLint + tsc
│   │   │   ├── runner.ts           # Playwright execution
│   │   │   └── vcs.ts              # GitHub/GitLab API
│   │   ├── models/
│   │   │   ├── script.ts
│   │   │   └── execution.ts
│   │   └── config.ts
│   ├── package.json
│   ├── tsconfig.json
│   └── Dockerfile
├── agent/
│   ├── playwright-mcp/
│   │   ├── chain.ts                # LangChain.js code gen chain
│   │   ├── prompts/
│   │   │   ├── codegen.txt
│   │   │   ├── pom.txt
│   │   │   └── troubleshoot.txt
│   │   ├── tools/
│   │   │   ├── eslint-tool.ts
│   │   │   └── tsc-tool.ts
│   │   └── selector-resolver.ts
│   └── tests/
│       └── codegen.test.ts
├── runner/
│   ├── Dockerfile                  # mcr.microsoft.com/playwright base
│   ├── playwright.config.ts
│   └── global-setup.ts
├── k8s/
│   ├── api-deployment.yaml
│   ├── runner-deployment.yaml
│   ├── service.yaml
│   ├── hpa.yaml
│   └── secrets.yaml
└── docker-compose.yaml
```

---

## 9. Security Considerations

| Concern | Mitigation |
|---|---|
| **Generated code execution** | Sandboxed Playwright runner containers; no host network access |
| **VCS credentials** | GitHub PAT stored in Key Vault; scoped to specific repos; rotated every 60 days |
| **Screenshot data** | Blob storage with SAS token expiry; no PII in test screenshots (staging data only) |
| **LLM code injection** | Generated code passes ESLint + TypeScript compilation before execution |
| **Authentication** | Entra ID OAuth 2.0; RBAC: Generator, Runner, Reviewer, Admin |
| **Network isolation** | Runner pods in dedicated subnet; egress limited to target URLs + blob storage |
| **Secrets in scripts** | Environment variables for auth tokens; never hardcoded in generated code |

---

## 10. Performance Targets

| Metric | Target | Notes |
|---|---|---|
| Script generation (single test case) | ≤ 15 s | Including LLM + lint + compile |
| Batch generation (10 test cases) | ≤ 60 s | Parallelised agent calls |
| Test execution (single browser) | ≤ 5 s per test | Playwright headless |
| Test execution (3 browsers) | ≤ 15 s per test | Parallel via workers |
| Troubleshoot diagnosis | ≤ 10 s | Screenshot + HTML analysis |
| VCS commit + PR | ≤ 5 s | GitHub API |
| API throughput | ≥ 100 req/s | At 3 API pods |
| Availability | 99.9 % | Multi-AZ deployment |

---

## 11. Implementation Roadmap

| Phase | Timeline | Deliverables |
|---|---|---|
| **Phase 1 — Core** | Weeks 1–2 | Script generation from Gherkin, basic .spec.ts output, Monaco editor |
| **Phase 2 — POM & Lint** | Weeks 3–4 | Page Object Model extraction, ESLint + tsc gate, selector strategies |
| **Phase 3 — Execution** | Weeks 5–6 | Playwright runner integration, multi-browser, trace/screenshots |
| **Phase 4 — Troubleshoot** | Weeks 7–8 | AI failure diagnosis, selector resolution, auto-patch |
| **Phase 5 — VCS & CI** | Weeks 9–10 | Git commit, PR creation, CI integration, Insights360 reporting |

---

📄 [Full Requirements Spec →](../solutions/auto-playpilot.md) · 🤖 [Agent Design →](../agents/playwright-mcp.md) · [← Platform Architecture](../architecture.md)
