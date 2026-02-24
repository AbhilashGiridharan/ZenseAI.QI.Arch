---
layout: default
title: "Test Strategy & Traceability"
description: "Platform-wide test strategy, traceability matrix, and coverage targets for ZenseAI.Qi."
---

[← Architecture Hub](../../index.md)
{: .fs-3 }

# Test Strategy & Traceability

---

## 1. Test Strategy Overview

ZenseAI.Qi follows a **layered test strategy** aligned with the test pyramid. Each solution owns its tests, while cross-cutting concerns (security, performance, AI guardrails) are centralised.

### Test Pyramid

```
         ┌─────────────┐
         │   E2E / UI   │  Playwright MCP — 7 solutions
         ├──────────────┤
         │ Integration  │  API ↔ Agent ↔ Provider contracts
         ├──────────────┤
         │    Unit      │  Jest / Vitest — business logic, transforms
         └──────────────┘
```

### Coverage Targets

| Layer | Coverage Target | Tool |
|---|---|---|
| Unit | ≥ 80 % line coverage | Jest / Vitest |
| Integration | All agent ↔ provider flows | Supertest + mocks |
| E2E | All P0 functional requirements | Playwright |
| Performance | All SLI endpoints | k6 |
| Security | OWASP Top 10 + SAST rules | semgrep + ZAP |

---

## 2. Test Types by Solution

| Solution | Unit | Integration | E2E | Performance | Security |
|---|---|---|---|---|---|
| DeepSpeci | ✅ | ✅ | ✅ | ✅ | ✅ |
| CaseGeni | ✅ | ✅ | ✅ | ✅ | ✅ |
| DataGeni | ✅ | ✅ | ✅ | ✅ | ✅ |
| Auto-PlayPilot | ✅ | ✅ | ✅ (meta) | ✅ | ✅ |
| Insights360 | ✅ | ✅ | ✅ | ✅ | ✅ |
| Perf-Xi | ✅ | ✅ | ✅ | ✅ (meta) | ✅ |
| Secure-Xi | ✅ | ✅ | ✅ | ✅ | ✅ (meta) |

---

## 3. Traceability Matrix

### DeepSpeci

| Requirement | Test Case(s) | Type | Agent |
|---|---|---|---|
| FR-DS-01 | TC-DS-01a: Upload PDF, TC-DS-01b: Parse JIRA | Integration | — |
| FR-DS-02 | TC-DS-02a: Ambiguity detection accuracy ≥ 85 % | E2E | Req Evaluator |
| FR-DS-03 | TC-DS-03a: Rewrite preserves intent | E2E | Req Evaluator |
| FR-DS-04 | TC-DS-04a: Completeness score correlation with SME | Unit | — |
| FR-DS-05 | TC-DS-05a: Testability score range 0-100 | Unit | — |

### CaseGeni

| Requirement | Test Case(s) | Type | Agent |
|---|---|---|---|
| FR-CG-01 | TC-CG-01a: Generate ≥ 1 test per requirement | E2E | Test Case Generator |
| FR-CG-03 | TC-CG-03a: BVA applied to numeric fields | Unit | — |
| FR-CG-04 | TC-CG-04a: Traceability matrix completeness = 100 % | Integration | — |
| FR-CG-05 | TC-CG-05a: Negative cases for auth flows | E2E | Test Case Generator |

### DataGeni

| Requirement | Test Case(s) | Type | Agent |
|---|---|---|---|
| FR-DG-01 | TC-DG-01a: Generated data matches schema | Integration | Test Data Generator |
| FR-DG-03 | TC-DG-03a: Zero PII in output | Security | Security Agent |
| FR-DG-04 | TC-DG-04a: FK constraints satisfied | Unit | — |
| FR-DG-05 | TC-DG-05a: 1 M rows in ≤ 120 s | Performance | Performance Agent |

### Auto-PlayPilot

| Requirement | Test Case(s) | Type | Agent |
|---|---|---|---|
| FR-AP-01 | TC-AP-01a: Generated .spec.ts compiles | Integration | Playwright MCP |
| FR-AP-03 | TC-AP-03a: ≥ 80 % data-testid selectors | Unit | — |
| FR-AP-04 | TC-AP-04a: Troubleshoot selector failure | E2E | Playwright MCP |
| FR-AP-05 | TC-AP-05a: Execute on staging, capture screenshots | E2E | Playwright MCP |

### Insights360

| Requirement | Test Case(s) | Type | Agent |
|---|---|---|---|
| FR-I3-01 | TC-I3-01a: Aggregate 3 solution results | Integration | Report Agent |
| FR-I3-02 | TC-I3-02a: AI summary ≤ 300 words | E2E | Report Agent |
| FR-I3-05 | TC-I3-05a: Drill-down from KPI to test | E2E | Playwright MCP |
| FR-I3-06 | TC-I3-06a: PDF export ≤ 10 s | Performance | — |

### Perf-Xi

| Requirement | Test Case(s) | Type | Agent |
|---|---|---|---|
| FR-PX-01 | TC-PX-01a: k6 script from OpenAPI compiles | Integration | Performance Agent |
| FR-PX-06 | TC-PX-06a: SLO breach triggers CI fail | E2E | Performance Agent |
| FR-PX-08 | TC-PX-08a: Capacity report produced | E2E | Performance Agent |

### Secure-Xi

| Requirement | Test Case(s) | Type | Agent |
|---|---|---|---|
| FR-SX-01 | TC-SX-01a: SAST finds known CWE-79 | E2E | Security Agent |
| FR-SX-02 | TC-SX-02a: DAST finds XSS on DVWA | E2E | Security Agent |
| FR-SX-03 | TC-SX-03a: Threat model has ≥ 6 entries | E2E | Security Agent |
| FR-SX-08 | TC-SX-08a: CI blocked on critical finding | Integration | Security Agent |

---

## 4. AI Agent Testing — Special Considerations

| Concern | Approach |
|---|---|
| Non-deterministic LLM output | Semantic similarity scoring (cosine ≥ 0.85) instead of exact match |
| Prompt injection | Adversarial test suite with 50 injection patterns |
| Token budget | Assert ≤ 8 000 tokens per request |
| PII leakage | Regex + NER scan on every prompt/response |
| Model fallback | Chaos test: primary model returns 500 → verify fallback activates |

---

## 5. Test Environment Strategy

| Environment | Purpose | Data |
|---|---|---|
| Local (dev) | Unit + integration tests | Synthetic fixtures |
| CI (GitHub Actions) | All automated tests | Synthetic + masked snapshots |
| Staging | E2E, performance, security | Full synthetic dataset |
| Production | Smoke tests only | Read-only queries |

---

## 6. Defect Taxonomy

| Severity | Definition | SLA (resolution) |
|---|---|---|
| Critical | System unusable; data loss | 4 hours |
| High | Major feature broken; workaround exists | 1 business day |
| Medium | Minor feature issue; cosmetic | 3 business days |
| Low | Enhancement; typo | Next sprint |
