---
layout: default
title: "Performance Specification"
description: "SLIs, SLOs, performance budgets, and runbooks for ZenseAI.Qi platform."
---

[← Architecture Hub](../../index.md)
{: .fs-3 }

# Performance Specification

---

## 1. Performance Philosophy

ZenseAI.Qi treats performance as a **first-class quality attribute**. Every solution publishes SLIs, enforces SLOs, and integrates with Perf-Xi for continuous measurement.

---

## 2. SLI / SLO Definitions

### Platform-Wide SLOs

| SLI | Target (SLO) | Measurement |
|---|---|---|
| API response time (p50) | ≤ 500 ms | Per-endpoint, sampled every minute |
| API response time (p95) | ≤ 2 000 ms | Per-endpoint, sampled every minute |
| API response time (p99) | ≤ 5 000 ms | Per-endpoint, sampled every minute |
| Error rate (5xx) | ≤ 0.5 % | Rolling 5-min window |
| Availability | ≥ 99.5 % | Monthly uptime (excl. maintenance) |
| Dashboard load time (p95) | ≤ 2 000 ms | Real-user monitoring |

### AI Agent SLOs

| SLI | Target (SLO) | Applies To |
|---|---|---|
| LLM call latency (p95) | ≤ 8 s | All agents |
| Token consumption per request | ≤ 8 000 tokens | All agents |
| Fallback activation time | ≤ 3 s | All agents |
| Agent total request time (p95) | ≤ 15 s | All agents |

### Solution-Specific SLOs

| Solution | SLI | Target |
|---|---|---|
| DeepSpeci | Requirement evaluation (p95) | ≤ 8 s |
| CaseGeni | Test case generation per req (p95) | ≤ 10 s |
| DataGeni | 1 K row generation (p95) | ≤ 5 s |
| DataGeni | 1 M row generation (p95) | ≤ 120 s |
| Auto-PlayPilot | Script generation per test (p95) | ≤ 12 s |
| Insights360 | Dashboard render (p95) | ≤ 2 s |
| Insights360 | PDF export | ≤ 10 s |
| Perf-Xi | k6 script generation (p95) | ≤ 10 s |
| Secure-Xi | SAST scan (100 KLOC) | ≤ 120 s |
| Secure-Xi | DAST scan | ≤ 300 s |

---

## 3. Performance Budget

| Resource | Budget | Enforcement |
|---|---|---|
| API payload size | ≤ 1 MB | API gateway rejects > 1 MB |
| Dashboard initial JS bundle | ≤ 250 KB (gzipped) | Webpack budget plugin |
| Dashboard LCP | ≤ 2.5 s | Lighthouse CI gate |
| DB query execution | ≤ 100 ms (p95) | Slow-query log alert |
| LLM prompt size | ≤ 4 000 tokens | Pre-send validation |
| LLM response size | ≤ 4 000 tokens | `max_tokens` parameter |

---

## 4. Load Profiles

### Standard Load

| Metric | Value |
|---|---|
| Concurrent users | 100 |
| Requests per second | 200 |
| Session duration | 15 min avg |
| Peak multiplier | 2× standard |

### Load Test Scenarios

| Scenario | Profile | Duration | Tool |
|---|---|---|---|
| Baseline | Constant 100 users | 10 min | k6 |
| Ramp-up | 10 → 200 users over 5 min | 15 min | k6 |
| Spike | 100 → 500 users in 30 s | 10 min | k6 |
| Soak | Constant 150 users | 2 hours | k6 |
| Stress | 100 → 1 000 users over 10 min | 20 min | k6 |

---

## 5. Capacity Planning

### Current Estimates (per environment)

| Component | Staging | Production |
|---|---|---|
| API servers | 2 pods (2 vCPU, 4 GB) | 4 pods (4 vCPU, 8 GB) |
| Worker nodes (agents) | 2 pods (4 vCPU, 8 GB) | 4 pods (4 vCPU, 8 GB) |
| Database | db.r6g.large | db.r6g.xlarge |
| Redis cache | cache.t3.medium | cache.r6g.large |
| LLM token budget/month | 5 M tokens | 20 M tokens |

### Scaling Triggers

| Metric | Threshold | Action |
|---|---|---|
| CPU utilisation (5-min avg) | > 70 % | Scale out +1 pod |
| Memory utilisation | > 80 % | Scale out +1 pod |
| Request queue depth | > 50 | Scale out +1 pod |
| LLM token usage (daily) | > 80 % of budget | Alert; throttle non-critical |
| DB connection pool | > 80 % | Alert; evaluate connection pooler |

---

## 6. Monitoring & Observability

### Metrics Pipeline

```
Application (OpenTelemetry SDK) → OTel Collector → Prometheus → Grafana dashboards
                                                 → Loki (logs)
                                                 → Tempo (traces)
```

### Key Dashboards

| Dashboard | Audience | Refresh |
|---|---|---|
| Platform SLO Overview | Platform team | 30 s |
| Agent Performance | AI/ML team | 1 min |
| Per-Solution Health | Solution owners | 1 min |
| LLM Cost Tracker | Engineering leadership | 5 min |

### Alerting Rules

| Alert | Condition | Severity | Channel |
|---|---|---|---|
| SLO breach (latency) | p95 > target for 5 min | Warning | Slack #perf |
| SLO breach (error rate) | 5xx > 0.5 % for 5 min | Critical | PagerDuty |
| SLO breach (availability) | Down > 1 min | Critical | PagerDuty |
| Token budget > 90 % | Daily consumption | Warning | Slack #platform |

---

## 7. Performance Testing Runbook

### Pre-Test Checklist

- [ ] Staging environment matches production topology
- [ ] Test data seeded (DataGeni synthetic dataset)
- [ ] Monitoring dashboards open
- [ ] Baseline results from last run available
- [ ] Stakeholders notified

### Execution Steps

1. Run **baseline** scenario (10 min, 100 users)
2. Validate all SLOs pass → proceed
3. Run **ramp-up** scenario
4. Run **spike** scenario
5. Run **soak** scenario (2 h)
6. Collect results; compare against baseline
7. Performance Agent generates regression report
8. Team reviews; file defects for regressions > 10 %

### Post-Test

- [ ] Results archived in Perf-Xi
- [ ] SLI data published to Insights360
- [ ] Regression defects filed (if any)
- [ ] Capacity-planning report updated

---

## 8. Performance Anti-Patterns to Avoid

| Anti-Pattern | Why It Matters | Guideline |
|---|---|---|
| N+1 queries | Latency scales linearly with data | Use eager loading / batch queries |
| Unbounded LLM prompts | Token costs & latency | Enforce 4 K token prompt cap |
| Synchronous LLM calls in request path | Blocks thread, high p95 | Use async worker queue |
| No caching of LLM responses | Redundant cost | Cache identical prompts (TTL 1 h) |
| Large JSON payloads | Serialisation overhead | Paginate; compress |
