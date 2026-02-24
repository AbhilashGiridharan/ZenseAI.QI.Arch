---
layout: default
title: "Solution: Auto-PlayPilot"
description: "Requirements specification for Auto-PlayPilot — AI-powered test automation with Playwright."
---

[← Back to Solutions](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# Auto-PlayPilot — Requirements Specification

---

## 1. Problem Statement & Goals

**Problem:** Converting test cases into automation scripts is manual, error-prone, and requires deep Playwright expertise. Maintaining scripts against evolving UIs is costly.

**Goals:**
- Auto-generate Playwright TypeScript scripts from structured test cases
- Achieve ≥ 70 % first-run pass rate on generated scripts
- Provide AI-assisted troubleshooting for failing scripts
- Reduce automation development effort by ≥ 40 %

---

## 2. In-Scope / Out-of-Scope

| In-Scope | Out-of-Scope |
|---|---|
| AI-generated Playwright scripts (.spec.ts) | Non-Playwright frameworks (Selenium, Cypress) |
| Page Object Model generation | Mobile-native automation |
| AI troubleshooting of failing scripts | Manual script authoring IDE |
| Test execution orchestration | CI/CD pipeline management |
| Visual regression (screenshot diff) | API-only testing (handled by backend tests) |
| Integration with CaseGeni (test cases) & DataGeni (fixtures) | |

---

## 3. Key Capabilities

| Capability | QE Work Lane | Supporting Agent(s) |
|---|---|---|
| Playwright script generation from test cases | AI-assisted automation workflows | Playwright MCP |
| Page Object Model creation | Create/optimise test libraries | Playwright MCP |
| AI troubleshooting of failures | Troubleshoot failing scripts | Playwright MCP |
| Test execution & reporting | AI-assisted automation workflows | Playwright MCP, Report Agent |
| Visual regression testing | Create/optimise test libraries | Playwright MCP |

---

## 4. Actors & Interfaces

| Actor | Interface | Human-in-the-Loop |
|---|---|---|
| Automation Engineer | Web UI — generate, review, execute scripts | Reviews generated code; approves execution |
| Technical Manager | Web UI — execution dashboards | Monitors pass/fail rates |
| Playwright MCP (Agent) | Internal API | Automated — code reviewed by engineer |

---

## 5. Architecture Summary

```
CaseGeni (test cases) + DataGeni (fixtures) → Auto-PlayPilot UI → API → Playwright MCP Agent
                                                                          ├── Model Provider (OpenAI primary)
                                                                          ├── ESLint + TypeScript compiler
                                                                          ├── Playwright Test Runner
                                                                          └── Scripts → VCS repo
                                                                                     → Execution results → Insights360
```

---

## 6. Functional Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| FR-AP-01 | Generate Playwright .spec.ts from structured test cases | Given Gherkin test cases, When generated, Then valid TypeScript that compiles |
| FR-AP-02 | Generate Page Object Models for new pages | Given an app URL, When analysed, Then POM classes with stable selectors |
| FR-AP-03 | Use data-testid selectors preferentially | Given generated scripts, Then ≥ 80 % selectors are data-testid |
| FR-AP-04 | AI troubleshoot failing tests | Given a failure log, When analysed, Then root cause + code fix suggested |
| FR-AP-05 | Execute tests against staging environment | Given approved scripts, When executed, Then results captured with screenshots |
| FR-AP-06 | Visual regression with screenshot comparison | Given baseline + current screenshots, Then pixel diff reported |
| FR-AP-07 | Commit scripts to VCS | Given approved scripts, When committed, Then pushed to configured repo/branch |
| FR-AP-08 | Integrate test fixtures from DataGeni | Given data fixtures, When used in scripts, Then data loaded via env/config |

### Gherkin Example (FR-AP-04)

```gherkin
Feature: AI Troubleshooting
  Scenario: Diagnose and fix a selector failure
    Given a Playwright test that fails with "locator not found: #submit-btn"
    When the Playwright MCP Agent analyses the failure
    Then the root cause is identified as "selector changed to [data-testid='submit']"
    And a code patch is suggested replacing "#submit-btn" with "[data-testid='submit']"
```

---

## 7. Non-Functional Requirements

| NFR | Target |
|---|---|
| Script generation latency (p95) | ≤ 12 s per test case |
| TypeScript compilation success | 100 % |
| First-run pass rate | ≥ 70 % |
| Selector stability (30-day) | ≥ 95 % |
| Concurrent executions | 20 parallel browser sessions |

---

## 8. Data Requirements

| Data | Direction | Schema |
|---|---|---|
| Structured test cases | Input | `{testId, gherkin, dataNeeds[], appUrl}` |
| App selector map | Input | `{page, selectors[{name, type, value}]}` |
| Generated scripts | Output | `.spec.ts` files |
| Execution results | Output | `{testId, status, duration, screenshots[], logs}` |

---

## 9. Test Strategy & Coverage

| Level | Approach | Agent |
|---|---|---|
| Unit | Vitest — code generation templates, selector logic | — |
| Integration | Auto-PlayPilot ↔ Playwright MCP ↔ Model Provider | — |
| E2E | Meta-test: generate scripts → execute → validate results | Playwright MCP |
| Performance | k6 — concurrent script generation requests | Performance Agent |
| Security | Code safety analysis (no eval, no secrets) | Security Agent |

---

## 10. Risks & Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| Generated code has runtime errors | Low first-run pass rate | TypeScript compile + lint gate before delivery |
| Selector instability | Flaky tests | Prefer data-testid; AI selector inference fallback |
| Unsafe code generated (eval, network calls) | Security risk | ESLint rules block unsafe patterns |

---

## 11. Roadmap Hooks

| Tag | Feature | Status |
|---|---|---|
| Apr-W1 | Basic Playwright script generation | TODO |
| Apr-W3 | Page Object Model generation | TODO |
| May-W2 | AI troubleshooting v1 | TODO |
| Jun-W1 | Visual regression | TODO |
| Jun-W3 | VCS integration (auto-commit) | TODO |
| Jul-W1 | DataGeni fixture integration | TODO |
