---
layout: default
title: "Secure-Xi — Solution Architecture & Implementation"
description: "Detailed architecture, component design, API contracts, data model, and implementation guide for Secure-Xi."
---

[← Back to Platform Architecture](../architecture.md) · [Requirements Spec](../solutions/secure-xi.md) · [Agent Design](../agents/security-agent.md)
{: .fs-3 }

# Secure-Xi — Solution Architecture & Implementation

---

## 1. Architecture Overview

<img src="../../assets/images/arch-secure-xi.svg" alt="Secure-Xi Architecture" width="100%" style="border-radius:6px; margin-bottom:1rem;" />

```mermaid
flowchart LR
    subgraph Inputs
        CODE[Code Repo]
        OAPI[OpenAPI Spec]
        IAC[Infra-as-Code]
    end

    subgraph Users
        SE([Security Engineer])
        TM([Technical Manager])
        DEV([Developer])
    end

    subgraph SecureXi["Secure-Xi"]
        UI[Secure-Xi UI<br/>Scans · Threat Model · Triage]
        API[Secure-Xi API]
    end

    subgraph Agent
        SA[Security Agent<br/>Threat Model · Triage · Test Gen]
    end

    subgraph Scanners
        SG[semgrep / CodeQL]
        ZAP[OWASP ZAP]
        SCA[Snyk / Trivy]
    end

    subgraph Models["Model Providers"]
        CLA[Claude]
        OAI[OpenAI]
    end

    DB[(PostgreSQL<br/>Findings · Threats · CVEs)]

    Inputs --> UI
    Users --> UI --> API --> SA
    SA --> CLA & OAI
    SA --> SG & ZAP & SCA
    SA -->|findings| API
    API --> DB

    API -.->|posture| I3[Insights360]
    API -.->|block| CI[CI Gate 🛑]
    API -.->|comments| PR[PR Comments]
```

### Key Flows

- SAST (semgrep/CodeQL), DAST (OWASP ZAP), and SCA (Snyk/Trivy) scans orchestrated
- The **Security Agent** generates STRIDE-based threat models from architecture docs
- Threat models auto-update on code/infra diffs
- AI severity triage: exploitability + impact assessment with confidence scores
- Critical/high findings block CI pipelines; medium findings posted as PR comments
- Security posture badge published to **Insights360** dashboard

---

## 2. Component Breakdown

```mermaid
flowchart TB
    subgraph FrontEnd["Secure-Xi UI (React / Next.js)"]
        SCAN_UI[Scan Manager<br/>SAST · DAST · SCA]
        THREAT[Threat Model Viewer<br/>STRIDE · Data Flow]
        TRIAGE[AI Triage Panel<br/>Severity · Exploitability]
        FINDINGS[Findings Dashboard<br/>CVEs · Remediation]
        POSTURE[Posture Overview<br/>Badge · Trends]
    end

    subgraph APILayer["Secure-Xi API (FastAPI)"]
        SCAN_SVC[Scan Orchestrator<br/>SAST + DAST + SCA dispatch]
        THREAT_SVC[Threat Model Service<br/>STRIDE generation]
        TRIAGE_SVC[Triage Service<br/>AI severity assessment]
        FINDING_SVC[Findings Service<br/>CVE dedup · tracking]
        CI_SVC[CI Gate Service<br/>Block / Allow decisions]
        PR_SVC[PR Comment Service<br/>GitHub / GitLab API]
    end

    subgraph AgentLayer["Security Agent"]
        STRIDE_A[STRIDE Threat Modeller]
        TRIAGE_A[AI Severity Triager]
        REMED_A[Remediation Advisor]
        TESTGEN_A[Security Test Generator]
    end

    subgraph DataLayer["Data Stores"]
        PG[(PostgreSQL<br/>Findings · Threats · Scans)]
        REDIS[(Redis<br/>Scan jobs · Cache)]
        VUL[(Vulnerability DB<br/>NVD · CVE mirror)]
    end

    FrontEnd --> APILayer
    APILayer --> AgentLayer
    APILayer --> DataLayer
```

| Component | Technology | Responsibility |
|---|---|---|
| **Secure-Xi UI** | React 18 / Next.js 15, Tailwind CSS | Scan management, threat model viewer, triage, posture |
| **Secure-Xi API** | Python 3.12, FastAPI, Uvicorn | Scan orchestration, threat models, triage, CI gate, PR comments |
| **Security Agent** | Python 3.12, LangChain/LangGraph | STRIDE modelling, AI triage, remediation, security test gen |
| **SAST Scanners** | semgrep / CodeQL | Static analysis (code patterns, injection, secrets) |
| **DAST Scanner** | OWASP ZAP | Dynamic analysis (XSS, SQLi, auth bypass) |
| **SCA Scanners** | Snyk / Trivy | Dependency vulnerability scanning |
| **PostgreSQL** | PostgreSQL 16 | Findings, threat models, scan history |
| **Redis** | Redis 7 | Async scan jobs, result caching |

---

## 3. Tech Stack

| Layer | Technology | Version | Purpose |
|---|---|---|---|
| Frontend | React + Next.js | 18.x / 15.x | Security dashboard SPA |
| Styling | Tailwind CSS | 3.x | Responsive UI |
| API Framework | FastAPI | 0.115+ | Async REST endpoints |
| Agent Framework | LangChain / LangGraph | 0.3+ | STRIDE, triage, remediation chains |
| LLM Providers | Claude, OpenAI | Latest | Threat modelling, severity triage |
| SAST | semgrep | 1.x | Code pattern analysis |
| SAST (alt) | CodeQL | 2.x | Semantic code analysis |
| DAST | OWASP ZAP | 2.15+ | Dynamic vulnerability scanning |
| SCA | Snyk CLI / Trivy | Latest / 0.56+ | Dependency vulnerability scanning |
| Database | PostgreSQL | 16 | Findings, threats, scans |
| Cache / Queue | Redis + Celery | 7.x / 5.x | Async scan jobs |
| VCS Integration | GitHub API / GitLab API | v4 | PR comments, webhook triggers |
| Containerisation | Docker | 24+ | Packaging |
| Orchestration | Kubernetes (AKS) | 1.29+ | Deployment |

---

## 4. API Contracts

### 4.1 Trigger Security Scan

```
POST /api/v1/securexi/scans
Content-Type: application/json

Request Body:
{
  "project_id": "uuid",
  "scan_types": ["sast", "dast", "sca"],
  "targets": {
    "repo_url": "https://github.com/org/app",
    "branch": "main",
    "openapi_url": "https://staging.app.com/openapi.json",
    "base_url": "https://staging.app.com"
  },
  "pr_number": 42,
  "ci_blocking": true
}

Response 202 Accepted:
{
  "scan_id": "uuid",
  "status": "running",
  "scan_types": ["sast", "dast", "sca"],
  "estimated_seconds": 120
}
```

### 4.2 Get Scan Results

```
GET /api/v1/securexi/scans/{scan_id}

Response 200 OK:
{
  "scan_id": "uuid",
  "status": "completed",
  "duration_ms": 95000,
  "summary": {
    "critical": 0,
    "high": 2,
    "medium": 5,
    "low": 12,
    "info": 8,
    "total": 27
  },
  "findings": [
    {
      "finding_id": "uuid",
      "scanner": "semgrep",
      "type": "sast",
      "rule": "python.lang.security.audit.eval-injection",
      "severity": "high",
      "file": "app/services/parser.py",
      "line": 42,
      "description": "Potential eval injection via user input",
      "cwe": "CWE-95",
      "ai_triage": {
        "exploitability": "high",
        "impact": "high",
        "confidence": 0.91,
        "recommendation": "Replace eval() with ast.literal_eval()"
      }
    }
  ],
  "ci_gate": "fail",
  "ci_gate_reason": "2 high-severity findings"
}
```

### 4.3 Generate Threat Model

```
POST /api/v1/securexi/threat-models
Content-Type: application/json

Request Body:
{
  "project_id": "uuid",
  "inputs": {
    "architecture_doc_url": "https://...",
    "repo_url": "https://github.com/org/app",
    "infra_as_code": "terraform" | "bicep" | "none"
  },
  "model_preference": "claude" | "openai" | "auto"
}

Response 202 Accepted:
{
  "threat_model_id": "uuid",
  "status": "generating"
}
```

### 4.4 Get Threat Model

```
GET /api/v1/securexi/threat-models/{id}

Response 200 OK:
{
  "threat_model_id": "uuid",
  "status": "completed",
  "stride": {
    "spoofing": [
      { "threat": "JWT token theft via XSS", "risk": "high", "mitigation": "Enable HttpOnly cookies" }
    ],
    "tampering": [...],
    "repudiation": [...],
    "information_disclosure": [...],
    "denial_of_service": [...],
    "elevation_of_privilege": [...]
  },
  "data_flows": [...],
  "trust_boundaries": [...],
  "model_used": "claude-sonnet-4-20250514",
  "confidence": 0.85
}
```

### 4.5 AI Triage Finding

```
POST /api/v1/securexi/findings/{finding_id}/triage
Content-Type: application/json

Request Body:
{
  "context": {
    "code_snippet": "...",
    "call_chain": ["main.py:10", "handler.py:25", "parser.py:42"],
    "reachable_from_input": true
  },
  "model_preference": "claude" | "openai" | "auto"
}

Response 200 OK:
{
  "finding_id": "uuid",
  "ai_triage": {
    "severity_override": "high",
    "exploitability": "high",
    "impact": "high",
    "confidence": 0.91,
    "reasoning": "User-controlled input reaches eval() via handler chain...",
    "recommendation": "Replace eval() with ast.literal_eval(); add input validation",
    "cwe": "CWE-95",
    "owasp": "A03:2021-Injection"
  },
  "model_used": "claude-sonnet-4-20250514"
}
```

### 4.6 Post PR Comment

```
POST /api/v1/securexi/scans/{scan_id}/pr-comment
Content-Type: application/json

Request Body:
{
  "repo": "org/app",
  "pr_number": 42,
  "severity_filter": ["medium", "high", "critical"],
  "include_remediation": true
}

Response 201 Created:
{
  "comment_id": 12345,
  "pr_url": "https://github.com/org/app/pull/42#issuecomment-12345",
  "findings_posted": 7
}
```

---

## 5. Data Model

```mermaid
erDiagram
    SCAN {
        uuid id PK
        uuid project_id FK
        json scan_types "['sast', 'dast', 'sca']"
        json targets "repo, branch, base_url"
        string status "running | completed | failed"
        int duration_ms
        int pr_number
        boolean ci_blocking
        string ci_gate "pass | fail"
        timestamp started_at
        timestamp completed_at
    }

    FINDING {
        uuid id PK
        uuid scan_id FK
        string scanner "semgrep | codeql | zap | snyk | trivy"
        string type "sast | dast | sca"
        string rule
        string severity "critical | high | medium | low | info"
        string file
        int line_number
        text description
        string cwe
        string owasp
        json ai_triage "exploitability, impact, confidence"
        string status "open | triaged | fixed | false_positive | accepted"
        timestamp created_at
    }

    THREAT_MODEL {
        uuid id PK
        uuid project_id FK
        json stride "6 STRIDE categories"
        json data_flows
        json trust_boundaries
        string model_used
        float confidence
        string status "generating | completed | failed"
        timestamp generated_at
    }

    REMEDIATION {
        uuid id PK
        uuid finding_id FK
        text recommendation
        text code_fix
        string status "suggested | applied | verified"
        timestamp created_at
    }

    PR_COMMENT {
        uuid id PK
        uuid scan_id FK
        string repo
        int pr_number
        int comment_id
        int findings_posted
        timestamp created_at
    }

    PROJECT {
        uuid id PK
        string name
        string tenant_id
    }

    SCAN ||--o{ FINDING : "produces"
    SCAN ||--o{ PR_COMMENT : "commented via"
    FINDING ||--o{ REMEDIATION : "remediated by"
    PROJECT ||--o{ SCAN : "scanned by"
    PROJECT ||--o{ THREAT_MODEL : "modelled by"
```

### Key Tables

| Table | Rows (est.) | Indexes |
|---|---|---|
| `finding` | ~100 K per tenant | `(scan_id, severity)`, `(status)`, `(cwe)` |
| `scan` | ~5 K per tenant | `(project_id, status)`, `(started_at)` |
| `threat_model` | ~200 per tenant | `(project_id)` |
| `remediation` | ~50 K per tenant | `(finding_id, status)` |

---

## 6. Integration Patterns

### 6.1 Upstream Integrations

| Source | Protocol | Details |
|---|---|---|
| **Code Repository** | GitHub / GitLab API | Clone repo for SAST scan; webhook on PR events |
| **OpenAPI Spec** | HTTP fetch | Target API spec for DAST scanning |
| **Infra-as-Code** | Git clone | Terraform / Bicep files for threat modelling |

### 6.2 Downstream Integrations

| Target | Protocol | Trigger | Payload |
|---|---|---|---|
| **Insights360** | Event bus (Redis pub/sub) | On scan complete | `{ scan_id, critical, high, posture }` |
| **CI Pipeline** | Webhook / REST | On CI-blocking scan with critical/high | `{ status: "fail", findings: [...] }` |
| **PR Comments** | GitHub / GitLab API | On scan with PR context | Markdown comment with findings + remediation |

### 6.3 Scan Orchestration Pipeline

```mermaid
flowchart TB
    TRIGGER[Scan Trigger<br/>PR hook / Manual / CI] --> DISPATCH[Scan Dispatcher]
    DISPATCH --> SAST[SAST<br/>semgrep / CodeQL]
    DISPATCH --> DAST[DAST<br/>OWASP ZAP]
    DISPATCH --> SCA[SCA<br/>Snyk / Trivy]
    SAST --> MERGE[Merge Findings<br/>Dedup by CWE + file + line]
    DAST --> MERGE
    SCA --> MERGE
    MERGE --> TRIAGE[AI Triage<br/>Security Agent]
    TRIAGE --> CLASSIFY[Classify<br/>Critical · High · Medium · Low]
    CLASSIFY --> GATE{CI Blocking?}
    GATE -->|Critical/High| BLOCK[CI Gate FAIL 🛑]
    GATE -->|Medium| COMMENT[PR Comment 💬]
    GATE -->|Low/Info| LOG[Log & Track]
    BLOCK --> REPORT[Publish to Insights360]
    COMMENT --> REPORT
    LOG --> REPORT
```

### 6.4 Threat Model Auto-Update

```
1. GitHub webhook fires on push / PR to main
2. Secure-Xi detects code or infra-as-code changes
3. Diff analysis: identify changed components / data flows
4. Security Agent re-evaluates affected STRIDE categories
5. Threat model updated incrementally (not full regeneration)
6. New threats flagged for security engineer review
```

---

## 7. Deployment Configuration

### 7.1 Kubernetes Resources

```yaml
# secure-xi-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: securexi-api
  namespace: zenseai-qi
spec:
  replicas: 2
  selector:
    matchLabels:
      app: securexi-api
  template:
    metadata:
      labels:
        app: securexi-api
    spec:
      containers:
        - name: securexi-api
          image: zenseai.azurecr.io/securexi-api:latest
          ports:
            - containerPort: 8005
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
                  name: securexi-secrets
                  key: database-url
            - name: REDIS_URL
              valueFrom:
                secretKeyRef:
                  name: securexi-secrets
                  key: redis-url
            - name: LLM_API_KEY
              valueFrom:
                secretKeyRef:
                  name: securexi-secrets
                  key: llm-api-key
            - name: GITHUB_TOKEN
              valueFrom:
                secretKeyRef:
                  name: securexi-secrets
                  key: github-token
---
# Scanner worker pods
apiVersion: apps/v1
kind: Deployment
metadata:
  name: securexi-scanner
  namespace: zenseai-qi
spec:
  replicas: 3
  selector:
    matchLabels:
      app: securexi-scanner
  template:
    metadata:
      labels:
        app: securexi-scanner
    spec:
      containers:
        - name: scanner
          image: zenseai.azurecr.io/securexi-scanner:latest
          command: ["celery", "-A", "app.scanner_worker", "worker", "--concurrency=2"]
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
| `DATABASE_URL` | PostgreSQL connection | `postgresql://user:pass@host:5432/securexi` |
| `REDIS_URL` | Redis (queue + cache) | `redis://host:6379/5` |
| `LLM_API_KEY` | Claude / OpenAI key | `key-...` |
| `GITHUB_TOKEN` | GitHub PAT for PR comments + repo access | `ghp_...` |
| `SEMGREP_APP_TOKEN` | semgrep registry token | `token-...` |
| `SNYK_TOKEN` | Snyk API token | `token-...` |
| `ZAP_API_KEY` | OWASP ZAP API key | `key-...` |
| `CI_WEBHOOK_URL` | CI pipeline callback | `https://ci.org/hooks/securexi` |
| `LOG_LEVEL` | Log verbosity | `INFO` |

### 7.3 Scaling Policy

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: securexi-scanner-hpa
  namespace: zenseai-qi
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: securexi-scanner
  minReplicas: 2
  maxReplicas: 10
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
secure-xi/
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── scans/              # Scan management
│   │   │   ├── threats/            # Threat model viewer
│   │   │   ├── triage/             # AI triage panel
│   │   │   ├── findings/           # Findings dashboard
│   │   │   └── posture/            # Security posture overview
│   │   ├── components/
│   │   │   ├── ScanProgress.tsx
│   │   │   ├── ThreatModelDiagram.tsx
│   │   │   ├── FindingCard.tsx
│   │   │   ├── SeverityBadge.tsx
│   │   │   └── PostureGauge.tsx
│   │   ├── hooks/
│   │   │   ├── useScans.ts
│   │   │   └── useThreats.ts
│   │   └── lib/
│   │       ├── api-client.ts
│   │       └── types.ts
│   ├── package.json
│   └── tsconfig.json
├── backend/
│   ├── app/
│   │   ├── main.py
│   │   ├── scanner_worker.py       # Celery scanner worker
│   │   ├── routers/
│   │   │   ├── scans.py            # Scan trigger + results
│   │   │   ├── threat_models.py    # STRIDE generation
│   │   │   ├── findings.py         # Finding CRUD + triage
│   │   │   ├── ci_gate.py          # CI blocking decisions
│   │   │   └── pr_comments.py      # GitHub/GitLab comments
│   │   ├── services/
│   │   │   ├── scan_orchestrator.py  # Dispatch SAST/DAST/SCA
│   │   │   ├── semgrep_runner.py
│   │   │   ├── zap_runner.py
│   │   │   ├── snyk_runner.py
│   │   │   ├── agent_client.py     # Security Agent calls
│   │   │   ├── triage.py           # AI severity assessment
│   │   │   ├── finding_dedup.py    # CWE + file + line dedup
│   │   │   └── pr_commenter.py     # Markdown comment builder
│   │   ├── models/
│   │   │   ├── scan.py
│   │   │   ├── finding.py
│   │   │   └── threat_model.py
│   │   ├── schemas/
│   │   │   ├── scan.py
│   │   │   └── finding.py
│   │   └── config.py
│   ├── alembic/
│   ├── tests/
│   │   ├── test_scan_orchestrator.py
│   │   ├── test_triage.py
│   │   ├── test_ci_gate.py
│   │   └── test_threat_model.py
│   ├── requirements.txt
│   └── Dockerfile
├── agent/
│   ├── security_agent/
│   │   ├── chain.py                # STRIDE + triage chain
│   │   ├── prompts/
│   │   │   ├── stride.txt
│   │   │   ├── triage.txt
│   │   │   ├── remediation.txt
│   │   │   └── security_test.txt
│   │   ├── tools/
│   │   │   ├── semgrep_tool.py
│   │   │   ├── zap_tool.py
│   │   │   └── snyk_tool.py
│   │   └── router.py
│   └── tests/
│       └── test_security_agent.py
├── k8s/
│   ├── api-deployment.yaml
│   ├── scanner-deployment.yaml
│   ├── service.yaml
│   ├── hpa.yaml
│   └── secrets.yaml
└── docker-compose.yaml
```

---

## 9. Security Considerations

| Concern | Mitigation |
|---|---|
| **Source code access** | Read-only Git clone; scoped deploy keys; code deleted after scan |
| **Scanner credentials** | semgrep/Snyk/ZAP tokens in Key Vault; rotated every 60 days |
| **Finding sensitivity** | Findings contain vulnerability details; encrypted at rest; tenant-isolated |
| **LLM data leakage** | Only code snippets (≤ 500 lines) sent to LLM for triage; no full codebase |
| **Authentication** | Entra ID OAuth 2.0; RBAC per tenant |
| **Authorisation** | Roles: Scanner, Triager, Remediator, Admin |
| **PR comment permissions** | GitHub token scoped to PR comments only; no write to code |
| **CI gate integrity** | Webhook signatures verified; idempotent gate decisions |
| **Scanner isolation** | Each scanner runs in isolated container; no cross-scan data |

---

## 10. Performance Targets

| Metric | Target | Notes |
|---|---|---|
| SAST scan (50 K LOC) | ≤ 90 s | semgrep parallel rules |
| DAST scan (50 endpoints) | ≤ 5 min | ZAP active scan |
| SCA scan (500 deps) | ≤ 30 s | Snyk/Trivy manifest analysis |
| Full scan (all 3 types) | ≤ 8 min | Parallelised scanners |
| AI triage (per finding) | ≤ 3 s | Including LLM round-trip |
| Threat model generation | ≤ 30 s | STRIDE analysis |
| PR comment post | ≤ 2 s | GitHub API call |
| API throughput | ≥ 80 req/s | At 2 API pods |
| Availability | 99.9 % | Multi-AZ deployment |

---

## 11. Implementation Roadmap

| Phase | Timeline | Deliverables |
|---|---|---|
| **Phase 1 — Core** | Weeks 1–2 | SAST scan (semgrep), findings dashboard, basic severity classification |
| **Phase 2 — DAST + SCA** | Weeks 3–4 | OWASP ZAP integration, Snyk/Trivy SCA, finding dedup |
| **Phase 3 — AI Triage** | Weeks 5–6 | Security Agent triage chain, exploitability scoring, remediation advisor |
| **Phase 4 — CI & PR** | Weeks 7–8 | CI gate integration, PR comments, webhook triggers |
| **Phase 5 — Threat Model** | Weeks 9–10 | STRIDE threat model generation, auto-update on diffs, posture badge |

---

📄 [Full Requirements Spec →](../solutions/secure-xi.md) · 🤖 [Agent Design →](../agents/security-agent.md) · [← Platform Architecture](../architecture.md)
