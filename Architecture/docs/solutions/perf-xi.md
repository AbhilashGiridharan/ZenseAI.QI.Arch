---
layout: default
title: "Solution: Perf-Xi"
description: "Requirements specification for Perf-Xi — AI-coordinated performance testing, SLI monitoring & observability."
---

[← Back to Solutions](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# Perf-Xi — Requirements Specification

---

## 1. Problem Statement & Goals

**Problem:** Performance test scripts are hand-crafted, SLI thresholds are buried in wikis, and there is no continuous feedback loop between test results and architectural decisions.

**Goals:**
- AI-generate k6/JMeter performance test scripts from API specs or user flows
- Define and track SLIs/SLOs per service endpoint
- Surface regressions within CI — fail builds when SLOs breach
- Provide capacity-planning recommendations via AI analysis

---

## 2. In-Scope / Out-of-Scope

| In-Scope | Out-of-Scope |
|---|---|
| AI-generated k6 scripts from OpenAPI specs | Infrastructure provisioning (IaC) |
| SLI/SLO definition & tracking | APM tool replacement (Datadog, New Relic) |
| Load profile configuration (ramp, soak, spike) | Chaos engineering |
| AI capacity-planning analysis | Hardware procurement |
| Integration with Insights360 (SLI overlay) | Database performance tuning |
| JMeter support (v2) | |

---

## 3. Key Capabilities

| Capability | QE Work Lane | Supporting Agent(s) |
|---|---|---|
| k6 script generation from OpenAPI | Performance test coordination | Performance Agent |
| SLI/SLO management | Performance test coordination | Performance Agent |
| Load profile orchestration | Performance test coordination | Performance Agent |
| Regression detection | Observability & feedback | Performance Agent, Report Agent |
| AI capacity planning | Performance test coordination | Performance Agent |

---

## 4. Actors & Interfaces

| Actor | Interface | Human-in-the-Loop |
|---|---|---|
| Performance Engineer | Web UI — script gen, SLO config, results | Reviews scripts; defines SLOs |
| Technical Manager | Dashboard — SLI gauges | Monitors trends |
| Performance Agent (AI) | Internal API | Automated; scripts reviewed by engineer |

---

## 5. Architecture Summary

```
OpenAPI spec / user flows → Perf-Xi UI → API → Performance Agent → Model Provider (Claude primary)
                                                    ├── k6 scripts → k6 Cloud / local runner
                                                    ├── SLI store (Postgres)
                                                    └── Results → Insights360
```

---

## 6. Functional Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| FR-PX-01 | Generate k6 scripts from OpenAPI spec | Given a valid spec, When generated, Then valid k6 JS that runs successfully |
| FR-PX-02 | Define SLIs per endpoint | Given an endpoint, When SLI defined (p50, p95, p99, error rate), Then persisted |
| FR-PX-03 | Define SLO targets per SLI | Given an SLI, When target set (e.g., p95 ≤ 500 ms), Then threshold enforced |
| FR-PX-04 | Configure load profiles | Given a scenario, When ramp/soak/spike configured, Then k6 `options` generated |
| FR-PX-05 | Execute load tests | Given approved scripts, When triggered, Then results captured (latency, throughput, errors) |
| FR-PX-06 | Detect SLO breaches | Given execution results, When SLO breached, Then alert raised + CI fail signal |
| FR-PX-07 | AI regression analysis | Given before/after results, When compared, Then root-cause hypothesis produced |
| FR-PX-08 | AI capacity-planning report | Given trend data (30d), When analysed, Then scaling recommendation produced |
| FR-PX-09 | Publish SLI data to Insights360 | Given SLI results, When published, Then visible on dashboard gauges |

### Gherkin Example (FR-PX-06)

```gherkin
Feature: SLO Breach Detection
  Scenario: p95 latency exceeds target
    Given SLO "POST /api/orders p95 ≤ 800 ms"
    And the latest test run shows p95 = 1240 ms
    When the Performance Agent evaluates results
    Then an SLO breach alert is raised
    And the CI pipeline receives a "fail" signal
    And the breach is recorded in the SLI history
```

---

## 7. Non-Functional Requirements

| NFR | Target |
|---|---|
| Script generation latency (p95) | ≤ 10 s |
| k6 script validity (compiles & runs) | 100 % |
| SLI data ingestion throughput | 10 000 data points/min |
| Dashboard SLI refresh | ≤ 30 s |
| Capacity-planning report generation | ≤ 20 s |
| Test result retention | 90 days |

---

## 8. Data Requirements

| Data | Direction | Schema |
|---|---|---|
| OpenAPI spec | Input | OpenAPI 3.x JSON/YAML |
| SLI definitions | Config | `{endpoint, method, sli: {metric, percentile}}` |
| SLO targets | Config | `{sliId, operator, threshold, unit}` |
| Load profiles | Config | `{scenario, stages[{target, duration}]}` |
| Execution results | Output | `{runId, endpoint, p50, p95, p99, errorRate, rps}` |
| Capacity report | Output | `{narrative, recommendations[], horizon}` |

---

## 9. Test Strategy & Coverage

| Level | Approach | Agent |
|---|---|---|
| Unit | Jest — SLI computation, threshold evaluation | — |
| Integration | Perf-Xi API ↔ Performance Agent ↔ k6 runner | — |
| E2E | Generate script → execute against mock API → validate SLI | Performance Agent |
| Meta-performance | k6 — Perf-Xi API itself under load | Performance Agent |
| Security | OWASP scan; secret-free k6 scripts | Security Agent |

---

## 10. Risks & Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Generated scripts miss auth headers | 401 errors, invalid results | Template includes auth pattern from spec `securitySchemes` |
| SLO targets too aggressive | False breach alerts | Baseline run auto-calibrates initial targets |
| Load test impacts production | Outage | Default to staging; production requires admin approval |

---

## 11. Roadmap Hooks

| Tag | Feature | Status |
|---|---|---|
| May-W1 | k6 script generation from OpenAPI | TODO |
| May-W3 | SLI/SLO definition UI | TODO |
| Jun-W1 | Load profile orchestration | TODO |
| Jun-W3 | SLO breach detection + CI integration | TODO |
| Jul-W1 | AI regression analysis | TODO |
| Jul-W3 | AI capacity-planning report | TODO |
