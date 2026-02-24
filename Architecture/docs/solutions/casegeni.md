---
layout: default
title: "Solution: CaseGeni"
description: "Requirements specification for CaseGeni — AI-powered test case generation."
---

[← Back to Solutions](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# CaseGeni — Requirements Specification

---

## 1. Problem Statement & Goals

**Problem:** Manual test case creation is time-consuming, inconsistent, and often misses edge cases, leading to incomplete test coverage and escaped defects.

**Goals:**
- Auto-generate structured test cases (Gherkin) from validated requirements
- Achieve ≥ 95 % requirement-to-test-case coverage
- Reduce test design effort by ≥ 50 %
- Maintain a deduplicated, reusable test library

---

## 2. In-Scope / Out-of-Scope

| In-Scope | Out-of-Scope |
|---|---|
| AI test case generation (positive, negative, boundary, edge) | Test execution (handled by Auto-PlayPilot) |
| Gherkin / structured format output | Test data generation (handled by DataGeni) |
| Deduplication against existing test library | Manual test case authoring UI (basic only) |
| Coverage matrix generation | Defect management |
| Priority / risk scoring per test case | |
| Export to Auto-PlayPilot & DataGeni | |

---

## 3. Key Capabilities

| Capability | QE Work Lane | Supporting Agent(s) |
|---|---|---|
| AI test case generation from requirements | Draft structured scenarios | Test Case Generator |
| Negative & edge-case scenario inference | Formulate testing approaches | Test Case Generator |
| Test library deduplication | Create/optimise test libraries | Test Case Generator |
| Coverage matrix (req → test case) | Formulate testing approaches | Test Case Generator, Report Agent |
| Risk-based priority scoring | Formulate testing approaches | Test Case Generator |

---

## 4. Actors & Interfaces

| Actor | Interface | Human-in-the-Loop |
|---|---|---|
| Automation Engineer | Web UI — generate, review, edit test cases | Reviews & approves AI-generated cases |
| Technical Manager | Web UI — coverage dashboard, approval workflow | Approves test plans |
| Developer | Web UI / API — view test cases linked to code | Reviews coverage for owned modules |
| Test Case Generator (Agent) | Internal API | Automated — outputs reviewed by humans |

---

## 5. Architecture Summary

```
DeepSpeci (validated reqs) → CaseGeni UI → CaseGeni API → Test Case Generator Agent
                                                            ├── Model Provider
                                                            └── Test cases → CaseGeni library
                                                                           → Auto-PlayPilot (automation)
                                                                           → DataGeni (data needs)
                                                                           → Insights360 (coverage metrics)
```

---

## 6. Functional Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| FR-CG-01 | Generate test cases from validated requirements | Given ≥ 1 validated req, When generated, Then ≥ 1 positive + 1 negative test case per req |
| FR-CG-02 | Output in Gherkin format | Given generated cases, Then each has valid Feature/Scenario/Given/When/Then syntax |
| FR-CG-03 | Deduplicate against existing library | Given new cases, When checked, Then duplicates (similarity ≥ 0.92) are flagged |
| FR-CG-04 | Coverage matrix | Given a requirement set, When generated, Then a matrix shows req ↔ test case mapping |
| FR-CG-05 | Priority / risk scoring | Given a test case, Then a priority (P1–P4) and risk score are assigned |
| FR-CG-06 | Export to Auto-PlayPilot | Given approved cases, When exported, Then Auto-PlayPilot receives structured JSON |
| FR-CG-07 | Export data dependencies to DataGeni | Given cases with data needs, When analysed, Then DataGeni receives data specs |
| FR-CG-08 | Manual edit & version history | Given a test case, When edited, Then changes are versioned |
| FR-CG-09 | Bulk generation for large requirement sets | Given > 50 requirements, When bulk-generated, Then cases are produced in batches |

### Gherkin Example (FR-CG-01)

```gherkin
Feature: Test Case Generation
  Scenario: Generate positive and negative cases
    Given a validated requirement "User can upload files up to 500 MB"
    When the Test Case Generator processes the requirement
    Then a positive test case is generated for a 400 MB file
    And a negative test case is generated for a 600 MB file
    And a boundary test case is generated for a 500 MB file
    And each test case has valid Gherkin syntax
```

---

## 7. Non-Functional Requirements

| NFR | Target |
|---|---|
| Generation latency (p95) | ≤ 8 s per requirement batch (up to 10 reqs) |
| Gherkin syntax validity | 100 % |
| Duplicate detection accuracy | ≥ 95 % |
| Availability | 99.9 % |
| Concurrent generation requests | 50 simultaneous |

---

## 8. Data Requirements

| Data | Direction | Schema |
|---|---|---|
| Validated requirements | Input | `{reqId, title, description, acceptanceCriteria[]}` |
| Test cases | Output | `{testId, reqId, gherkin, priority, riskScore, dataNeeds[]}` |
| Coverage matrix | Output | `{reqId, testCaseIds[], coveragePercent}` |
| Test library | Internal | Persistent store of all generated & approved test cases |

---

## 9. Test Strategy & Coverage

| Level | Approach | Agent |
|---|---|---|
| Unit | Vitest — Gherkin parser, dedup algorithm | — |
| Integration | CaseGeni ↔ Test Case Generator ↔ Model Provider | — |
| E2E | Playwright — generate → review → export flow | Playwright MCP |
| Performance | k6 — bulk generation (100 reqs) | Performance Agent |
| Security | SAST on CaseGeni API | Security Agent |

---

## 10. Risks & Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Low edge-case detection rate | Missed defects | Golden-set regression; continuous eval improvement |
| Dedup false negatives | Bloated test library | Periodic library audit (manual + AI) |
| Invalid Gherkin syntax | Automation failures downstream | Parser validation gate before save |

---

## 11. Roadmap Hooks

| Tag | Feature | Status |
|---|---|---|
| Mar-W2 | Basic test case generation | TODO |
| Apr-W1 | Deduplication engine | TODO |
| Apr-W3 | Coverage matrix | TODO |
| May-W1 | Risk-based prioritisation | TODO |
| May-W3 | Bulk generation | TODO |
| Jun-W2 | Export to Auto-PlayPilot & DataGeni | TODO |
