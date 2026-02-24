---
layout: default
title: "Agent: Performance Agent"
description: "Design spec for the Performance Agent AI Agent."
---

[← Back to Agents](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# Performance Agent — Agent Design Spec

---

## 1. Mission

Coordinate performance test execution, analyse results against SLIs/SLOs, detect regressions, and provide AI-driven root-cause analysis and optimisation recommendations.

**QE Work Lane:** Coordinate performance test execution; performance observability.

---

## 2. Inputs & Outputs

| Direction | Artifact | Format |
|---|---|---|
| **Input** | Performance test plan (endpoints, scenarios, load profiles) | YAML / JSON |
| **Input** | SLI/SLO definitions | YAML |
| **Input** | Test data volumes (from Test Data Generator) | CSV / JSON |
| **Input** | Historical performance baselines | JSON (time-series) |
| **Output** | k6 / JMeter test scripts | `.js` / `.jmx` files |
| **Output** | Execution results (latency, throughput, error rate) | JSON / Prometheus |
| **Output** | Regression analysis report | Markdown |
| **Output** | Root-cause analysis & recommendations | Markdown |

---

## 3. LLM / Model Usage

| Model | Use Case | Selection Logic |
|---|---|---|
| **OpenAI** | k6 script generation, structured analysis | Primary — code + data analysis |
| **Claude** | Root-cause narrative, deep analysis | Complex multi-factor regressions |
| **Gemini** | Chart / graph interpretation | Visual performance dashboard analysis |

---

## 4. Orchestration

```
Perf-Xi UI → /api/agents/performance-agent/plan
               ├── Define load profiles & scenarios
               ├── Generate k6 scripts
               └── Return to Perf-Xi → engineer review

Perf-Xi UI → /api/agents/performance-agent/execute
               ├── Trigger test execution (k6 cloud / local)
               ├── Collect metrics in real-time
               └── Stream to Perf-Xi dashboard

Perf-Xi UI → /api/agents/performance-agent/analyse
               ├── Compare results vs. SLOs & baselines
               ├── Call LLM → root-cause analysis
               ├── Generate recommendations
               └── Return to Perf-Xi → engineer review → Insights360
```

- **Caller:** Perf-Xi (primary), CI/CD pipeline (pre-deploy gate)
- **Consumers:** Perf-Xi (dashboards), Insights360 (aggregated metrics), CI/CD (pass/fail gate)

---

## 5. Guardrails

| Guardrail | Implementation |
|---|---|
| **Blast radius** | Max concurrent VUs configurable; hard ceiling at 1 000 |
| **Environment safety** | Only run against staging/perf environments; block production |
| **Determinism** | Fixed random seed for load distribution |
| **Data isolation** | Performance test data does not pollute production DB |
| **Cost control** | Cloud execution budget cap per run; alert at 80 % |
| **Timeout** | Max test duration: 30 min; auto-abort on exceeded |

---

## 6. APIs / Tools

| Tool | Purpose |
|---|---|
| k6 (Grafana) | Load test execution engine |
| Prometheus / Grafana | Metrics collection & visualisation |
| Application Insights | Server-side telemetry correlation |
| CI/CD pipeline API | Trigger / gate on perf results |
| Historical baselines DB | Store & retrieve past results for comparison |

---

## 7. Evaluation

| KPI | Target | Method |
|---|---|---|
| SLO breach detection accuracy | ≥ 95 % | Compare AI detection vs. manual review |
| Regression detection (p95 latency) | ≥ 90 % | Golden set of known regressions |
| Root-cause recommendation relevance | ≥ 70 % accepted | Track engineer acceptance rate |
| Script generation correctness | ≥ 90 % run without errors | Execute generated k6 scripts |
| Analysis latency | ≤ 20 s | Post-execution analysis time |

---

## 8. Failure Modes & Fallbacks

| Failure Mode | Fallback |
|---|---|
| k6 execution failure | Retry once; fall back to reduced load profile |
| Metrics pipeline down | Buffer locally; replay when restored |
| LLM analysis timeout | Return raw metrics without narrative |
| SLO definition missing | Use platform defaults; warn engineer |
| Cost ceiling hit mid-test | Graceful ramp-down; report partial results |
