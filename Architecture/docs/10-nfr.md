---
layout: default
title: "10. Non-Functional Requirements"
nav_order: 12
description: "Performance targets, availability, RTO/RPO, scalability, and cost constraints."
permalink: "/docs/nfr/"
---

[← Back to Architecture Hub]({{ '/' | relative_url }})
{: .fs-3 }

# 10. Non-Functional Requirements

---

## Requirements Matrix

| Requirement | Target |
|---|---|
| **Response latency (p95)** | ≤ 3 s end-to-end |
| **Availability / SLA** | 99.9 % (monthly) |
| **RTO** | ≤ 1 h |
| **RPO** | ≤ 15 min |
| **Throughput** | 500 concurrent users sustained |
| **Index size** | ≤ 50 GB (S1 AI Search) |
| **Token budget** | ≤ 500 K tokens/day |
| **Cold start** | ≤ 10 s (container pull + warm-up) |
| **Cost ceiling** | Defined per environment in Bicep params |
| **Scalability** | HPA 2–20 pods per service; AKS cluster autoscaler 3–10 nodes |

---

## Performance Breakdown

| Component | p50 Target | p95 Target |
|---|---|---|
| UI render (first paint) | 200 ms | 500 ms |
| API Gateway overhead | 10 ms | 30 ms |
| Retrieval (hybrid search) | 150 ms | 400 ms |
| LLM completion (first token) | 300 ms | 800 ms |
| LLM completion (full) | 1 s | 2.5 s |
| **Total E2E** | **1.5 s** | **3 s** |

---

## Resilience

- **Circuit breaker** on LLM calls (fallback to cached response or graceful degradation message).
- **Retry policy**: exponential backoff (max 3 retries, jitter).
- **Health checks**: liveness + readiness probes on every pod.
- **Multi-zone AKS** in production (3 availability zones).

---

**Previous:** [← Deployment & Environments]({{ '/docs/deployment/' | relative_url }}) · **Next:** [Appendix →]({{ '/docs/appendix/' | relative_url }})
