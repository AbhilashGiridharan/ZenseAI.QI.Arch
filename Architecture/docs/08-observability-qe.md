---
layout: default
title: "8. Observability & Quality Engineering"
nav_order: 10
description: "Logging, tracing, metrics, synthetic tests, and AI test automation agent for QI Offerings."
permalink: "/docs/observability-qe/"
---

[← Back to Architecture Hub]({{ '/' | relative_url }})
{: .fs-3 }

# 8. Observability & Quality Engineering (QE)

---

## Logging & Tracing

- **OpenTelemetry SDK** instrumented in every FastAPI service.
- Traces exported to **Azure Monitor Application Insights** (W3C trace-context).
- Structured JSON logs → Azure Log Analytics workspace.
- Correlation ID propagated from UI → Gateway → all services.

---

## Key Metrics

| Metric | Source | Target |
|---|---|---|
| E2E latency (p50 / p95) | App Insights | ≤ 1.5 s / ≤ 3 s |
| Token usage (prompt + completion) | OpenAI SDK callback | Budget ≤ 500 K tokens/day |
| Retrieval precision@5 | Offline eval job | ≥ 0.85 |
| Answer confidence (mean) | Orchestration Service | ≥ 0.75 |
| Error rate (5xx) | APIM analytics | < 0.5 % |
| Ingestion throughput | Service Bus metrics | ≥ 100 docs/hr |

---

## Quality Engineering

- **Synthetic E2E tests** – Playwright test suite against staging (runs nightly).
- **AI test-data agent** – generates realistic question variants from golden set using GPT-4.1.
- **Performance tests** – k6 load tests simulating 200 concurrent users; run pre-deploy.
- **Retrieval regression** – weekly comparison of precision/recall against baseline.

---

## Test Automation Agent Sequence

```mermaid
sequenceDiagram
    actor QE as QE Engineer
    participant Agent as AI Test Agent
    participant App as QI Offerings API
    participant DB as PostgreSQL
    participant Dash as Grafana Dashboard

    QE->>Agent: Trigger test plan generation
    Agent->>Agent: Generate test cases from golden set
    loop For each test case
        Agent->>App: POST /api/ask {query, expected}
        App-->>Agent: {answer, citations, confidence}
        Agent->>Agent: Evaluate (BLEU, semantic sim, citation match)
    end
    Agent->>DB: Store evaluation results
    Agent->>Dash: Push metrics (precision, recall, confidence)
    Agent-->>QE: Summary report + regressions
```

---

**Previous:** [← Security & Compliance]({{ '/docs/security-compliance/' | relative_url }}) · **Next:** [Deployment & Environments →]({{ '/docs/deployment/' | relative_url }})
