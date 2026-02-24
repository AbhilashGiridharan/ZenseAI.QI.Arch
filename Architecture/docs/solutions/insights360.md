---
layout: default
title: "Solution: Insights360"
description: "Requirements specification for Insights360 — consolidated test reporting, analytics & executive dashboards."
---

[← Back to Solutions](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# Insights360 — Requirements Specification

---

## 1. Problem Statement & Goals

**Problem:** Quality metrics are scattered across execution logs, JIRA tickets, spreadsheets, and chat threads. Stakeholders lack a single pane of glass to track release readiness.

**Goals:**
- Aggregate execution results, defects, coverage, and risk signals into one dashboard
- Provide AI-generated release-readiness summaries
- Enable drill-down from executive KPIs to individual test results
- Deliver daily digest notifications (email, Slack/Teams)

---

## 2. In-Scope / Out-of-Scope

| In-Scope | Out-of-Scope |
|---|---|
| Unified dashboard: pass/fail, coverage, defects | Test execution engine (handled by Auto-PlayPilot) |
| AI narrative summaries per release | Test case authoring |
| Trend analytics (7d, 30d, release-over-release) | Security scanning execution |
| Export (PDF, CSV) | Performance test execution |
| Notification webhooks (Slack, Teams, email) | Custom BI tool connectors (v2) |
| Data from all 6 sibling solutions | |

---

## 3. Key Capabilities

| Capability | QE Work Lane | Supporting Agent(s) |
|---|---|---|
| Test result aggregation | Interpret & validate requirements (coverage view) | Report Agent |
| AI-generated release summary | AI-assisted automation workflows | Report Agent |
| Defect trend analysis | Troubleshoot failing scripts | Report Agent |
| Performance SLI overlay | Performance test coordination | Report Agent, Performance Agent |
| Security posture badge | Vulnerability scanning & threat analysis | Report Agent, Security Agent |

---

## 4. Actors & Interfaces

| Actor | Interface | Human-in-the-Loop |
|---|---|---|
| Manager | Web dashboard — KPIs, summaries | Reviews & approves release summary |
| Technical Manager | Web dashboard — drill-down to tests | Filters by component/sprint |
| Non-Technical User | Email/Slack digest | Reads summary |
| Report Agent (AI) | Internal API — data → narrative | Automated; summary reviewed by Manager |

---

## 5. Architecture Summary

```
Auto-PlayPilot results ─┐
CaseGeni coverage      ─┤
DataGeni data health   ─┤─→ Insights360 API → Report Agent → Model Provider (Gemini primary)
Perf-Xi metrics        ─┤                                     │
Secure-Xi findings     ─┘                                     └─→ Dashboards / Notifications
```

---

## 6. Functional Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| FR-I3-01 | Aggregate test results from all solutions | Given execution data, When aggregated, Then unified pass/fail/skip counts displayed |
| FR-I3-02 | Generate AI release-readiness narrative | Given aggregated data, When summarised, Then ≤ 300-word narrative produced |
| FR-I3-03 | Display coverage heatmap | Given test-case ↔ requirement mapping, Then heatmap shows covered/uncovered areas |
| FR-I3-04 | Trend charts (7d, 30d, release) | Given historical data, Then sparklines and trend arrows rendered |
| FR-I3-05 | Drill-down from KPI to individual test | Given a KPI tile, When clicked, Then filtered test list shown |
| FR-I3-06 | Export dashboard as PDF/CSV | Given current dashboard, When exported, Then downloadable file produced within 10 s |
| FR-I3-07 | Send daily digest notification | Given a configured channel, When triggered daily at 08:00, Then summary sent |
| FR-I3-08 | Defect-to-test linkage | Given JIRA defect IDs, When correlated, Then failing tests linked to defects |
| FR-I3-09 | Performance SLI overlay on dashboard | Given Perf-Xi SLI data, Then gauges rendered next to test results |

### Gherkin Example (FR-I3-02)

```gherkin
Feature: AI Release-Readiness Summary
  Scenario: Generate summary for Sprint 12
    Given execution results: 420 passed, 18 failed, 3 skipped
    And 2 critical defects open
    And performance SLI "p95 login" = 980 ms (target ≤ 1000 ms)
    When the Report Agent generates a release summary
    Then a ≤ 300-word narrative is produced
    And it flags 2 critical defects as blockers
    And confidence level is "AMBER"
```

---

## 7. Non-Functional Requirements

| NFR | Target |
|---|---|
| Dashboard load time (p95) | ≤ 2 s |
| Data freshness | ≤ 5 min lag |
| Concurrent users | 100 |
| Report generation latency (p95) | ≤ 8 s |
| PDF export | ≤ 10 s |
| Notification delivery | ≤ 60 s |
| Uptime | 99.5 % |

---

## 8. Data Requirements

| Data | Direction | Schema |
|---|---|---|
| Execution results | Input | `{runId, tests[{id, status, duration, error?}]}` |
| Coverage map | Input | `{requirementId, testIds[]}` |
| Performance SLIs | Input | `{metric, value, target, unit}` |
| Security findings | Input | `{vulnId, severity, status}` |
| AI summary | Output | `{narrative, confidence, blockers[], generated}` |
| Dashboard state | Output | `{filters, tiles[{type, data}]}` |

---

## 9. Test Strategy & Coverage

| Level | Approach | Agent |
|---|---|---|
| Unit | Jest — aggregation logic, chart data transforms | — |
| Integration | Insights360 API ↔ Report Agent ↔ Model Provider | — |
| E2E | Playwright — dashboard renders, drill-down, export | Playwright MCP |
| Performance | k6 — 100 concurrent dashboard loads | Performance Agent |
| Security | OWASP scan on dashboard (XSS, CSRF) | Security Agent |

---

## 10. Risks & Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Stale data displayed | Misleading KPIs | Show "last-updated" timestamp; alert if lag > 10 min |
| AI summary hallucination | Incorrect release call | Confidence badge + manager review gate |
| Dashboard performance under load | Slow UX | Materialised views; CDN for static assets |

---

## 11. Roadmap Hooks

| Tag | Feature | Status |
|---|---|---|
| May-W1 | MVP dashboard (pass/fail/skip) | TODO |
| May-W3 | AI release narrative | TODO |
| Jun-W1 | Trend analytics | TODO |
| Jun-W3 | Notification integration (Slack/Teams) | TODO |
| Jul-W1 | PDF/CSV export | TODO |
| Jul-W3 | Perf-Xi + Secure-Xi overlays | TODO |
