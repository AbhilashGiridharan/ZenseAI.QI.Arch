---
layout: default
title: "Agent: Report Agent"
description: "Design spec for the Report Agent AI Agent."
---

[← Back to Agents](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# Report Agent — Agent Design Spec

---

## 1. Mission

Aggregate test results, metrics, and quality signals across all QI Workspace solutions into consolidated, actionable reports and executive dashboards.

**QE Work Lane:** Formulate testing approaches (reporting view); performance observability (metrics aggregation).

---

## 2. Inputs & Outputs

| Direction | Artifact | Format |
|---|---|---|
| **Input** | Test execution results (all solutions) | JSON |
| **Input** | Coverage matrices (CaseGeni) | CSV / JSON |
| **Input** | Performance metrics (Perf-Xi) | JSON / Prometheus |
| **Input** | Security findings (Secure-Xi) | JSON / SARIF |
| **Input** | Requirement traceability (DeepSpeci) | JSON |
| **Output** | Executive summary report | Markdown / PDF |
| **Output** | Interactive dashboard data | JSON (Grafana-compatible) |
| **Output** | Trend analysis & recommendations | Markdown |
| **Output** | Release readiness scorecard | JSON / Markdown |

---

## 3. LLM / Model Usage

| Model | Use Case | Selection Logic |
|---|---|---|
| **Claude** | Long-form executive summaries, narrative insights | Primary for reports |
| **OpenAI** | Structured data summarisation, JSON output | Data-heavy aggregations |
| **Gemini** | Multi-modal: chart interpretation, screenshot analysis | Visual report elements |

---

## 4. Orchestration

```
Insights360 UI → /api/agents/report-agent/generate
                    ├── Collect data from all solution APIs
                    ├── Aggregate & normalise metrics
                    ├── Call LLM → generate narrative + recommendations
                    ├── Compose report (template + AI content)
                    └── Return to Insights360 → Manager review

Scheduled (nightly / weekly):
  CRON → /api/agents/report-agent/scheduled
           ├── Pull latest results from all solutions
           ├── Generate trend report
           └── Push to Insights360 dashboard + notify stakeholders
```

- **Caller:** Insights360 (primary), all solutions (event-driven on test completion)
- **Consumers:** Insights360 (dashboards), Managers (email reports), CI/CD (release gate)

---

## 5. Guardrails

| Guardrail | Implementation |
|---|---|
| **Data accuracy** | Cross-verify counts against source APIs; flag discrepancies |
| **Hallucination** | No fabricated metrics; all numbers sourced from data |
| **PII** | Strip user identifiers from reports; use role-based aggregation |
| **Bias** | Balanced presentation of pass/fail; no suppression of failures |
| **Freshness** | Reports include data timestamp; stale data (> 24 h) flagged |

---

## 6. APIs / Tools

| Tool | Purpose |
|---|---|
| All solution APIs | Aggregate test results, coverage, metrics |
| Grafana API | Push dashboard panels |
| Email / Teams webhook | Distribute reports |
| PDF renderer (Puppeteer) | Generate PDF exports |
| Template engine | Compose consistent report layouts |

---

## 7. Evaluation

| KPI | Target | Method |
|---|---|---|
| Data accuracy (vs. source) | 100 % match | Automated reconciliation |
| Report generation latency | ≤ 30 s | End-to-end timer |
| Stakeholder satisfaction | ≥ 4.0 / 5.0 | Quarterly survey |
| Recommendation actionability | ≥ 70 % acted upon | Track follow-up actions |
| Trend prediction accuracy | ≥ 75 % | Compare predicted vs. actual next-sprint defects |

---

## 8. Failure Modes & Fallbacks

| Failure Mode | Fallback |
|---|---|
| Source API unavailable | Use last-cached data; mark as stale in report |
| LLM fails to generate narrative | Fall back to template-only report (no AI summary) |
| Data discrepancy detected | Include warning banner; log for investigation |
| PDF render fails | Serve Markdown version; retry PDF async |
| Scheduled job misfire | Retry within 1 h window; alert on repeated failure |
