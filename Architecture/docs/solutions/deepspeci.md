---
layout: default
title: "Solution: DeepSpeci"
description: "Requirements specification for DeepSpeci — AI-powered requirement analysis and validation."
---

[← Back to Solutions](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# DeepSpeci — Requirements Specification

---

## 1. Problem Statement & Goals

**Problem:** Business requirements are often ambiguous, incomplete, and lack testability criteria, causing downstream test-design rework and missed coverage.

**Goals:**
- Reduce requirement ambiguity by ≥ 60 % through AI-assisted validation
- Automatically generate testability scores for every requirement
- Provide requirement-to-test traceability from day one
- Enable non-technical users to input requirements in natural language

---

## 2. In-Scope / Out-of-Scope

| In-Scope | Out-of-Scope |
|---|---|
| Natural-language requirement intake | Requirements authoring from scratch (AI assists, not replaces) |
| AI validation (gap detection, ambiguity flagging) | Project management features (sprints, boards) |
| Testability scoring | Integration with non-supported ALM tools |
| Requirement traceability matrix generation | Regulatory compliance certification |
| Import from JIRA, Confluence, DOCX, Markdown | Real-time collaborative editing |
| Export validated requirements to CaseGeni | |

---

## 3. Key Capabilities

| Capability | QE Work Lane | Supporting Agent(s) |
|---|---|---|
| Requirement ingestion & parsing | Requirements interpretation | Requirement Evaluator |
| Ambiguity detection & resolution | Requirements validation | Requirement Evaluator |
| Testability scoring | Formulate testing approaches | Requirement Evaluator |
| Gap analysis & completeness check | Requirements interpretation | Requirement Evaluator |
| Traceability matrix generation | Requirements validation | Requirement Evaluator, Report Agent |
| Natural-language requirement input | Requirements interpretation | Requirement Evaluator |

---

## 4. Actors & Interfaces

| Actor | Interface | Human-in-the-Loop |
|---|---|---|
| Non-Technical User | Web UI — natural-language input, review dashboard | Approves/edits AI suggestions |
| Technical Manager | Web UI — validation dashboard, traceability view | Signs off validated requirements |
| SME | Web UI — review flagged ambiguities | Resolves domain-specific ambiguities |
| Developer | Web UI / API — requirement detail view | Reviews linked code artefacts |
| Requirement Evaluator (Agent) | Internal API | Automated — outputs reviewed by humans |

---

## 5. Architecture Summary

```
DeepSpeci sits within QI Workspace as the entry point for all requirements.

User → DeepSpeci UI → DeepSpeci API → Requirement Evaluator Agent
                                        ├── Model Provider (Gemini/Claude/OpenAI)
                                        └── Validated output → DeepSpeci → CaseGeni
                                                             → Insights360 (metrics)
```

**Dependencies:** Requirement Evaluator (primary agent), Model Providers (Gemini, Claude, OpenAI), CaseGeni (downstream consumer), Insights360 (reporting).

---

## 6. Functional Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| FR-DS-01 | Import requirements from JIRA, Confluence, DOCX, Markdown | Given a JIRA project, When I click "Import", Then all user stories are parsed and displayed |
| FR-DS-02 | Natural-language requirement input via chat interface | Given the chat UI, When I type a requirement in plain English, Then it is parsed into structured format |
| FR-DS-03 | AI-powered ambiguity detection | Given a requirement, When analysed, Then ambiguous terms are highlighted with suggestions |
| FR-DS-04 | Testability score (0–1) for each requirement | Given a requirement, When scored, Then a testability value and reasoning are shown |
| FR-DS-05 | Gap analysis — missing acceptance criteria, preconditions | Given a requirement set, When analysed, Then gaps are listed with severity |
| FR-DS-06 | Requirement traceability matrix | Given validated requirements, When generated, Then each req maps to test cases (from CaseGeni) |
| FR-DS-07 | Export validated requirements to CaseGeni | Given validated reqs, When exported, Then CaseGeni receives structured JSON |
| FR-DS-08 | Version history for requirement changes | Given a requirement, When edited, Then previous versions are accessible |
| FR-DS-09 | Human review workflow (approve / revise / reject) | Given AI output, When reviewed, Then user can approve, revise, or reject each item |

### Gherkin Example (FR-DS-03)

```gherkin
Feature: Ambiguity Detection
  Scenario: Detect ambiguous requirement language
    Given a requirement "The system should handle large files"
    When the Requirement Evaluator analyses the requirement
    Then the word "large" is flagged as ambiguous
    And a suggestion is provided: "Specify file size limit (e.g., ≤ 500 MB)"
    And the ambiguity confidence score is ≥ 0.7
```

---

## 7. Non-Functional Requirements

| NFR | Target |
|---|---|
| Response latency (p95) | ≤ 5 s per requirement validation |
| Availability | 99.9 % monthly |
| Concurrent users | 100 simultaneous |
| Data encryption | AES-256 at rest, TLS 1.3 in transit |
| PII handling | Redact before LLM; Presidio scan |
| Audit logging | All actions logged with user, timestamp, outcome |
| Observability | OpenTelemetry traces, Grafana dashboards via Insights360 |

---

## 8. Data Requirements

| Data | Direction | Schema | PII |
|---|---|---|---|
| Raw requirements | Input | `{id, title, description, source, priority}` | Possible — redact names, emails |
| Validated requirements | Output | `{id, title, description, testabilityScore, gaps[], ambiguities[]}` | Redacted |
| Traceability matrix | Output | `{reqId, testCaseIds[], coverage}` | None |
| Audit log | Internal | `{userId, action, timestamp, reqId, outcome}` | userId anonymised in exports |

---

## 9. Test Strategy & Coverage

| Level | Approach | Agent |
|---|---|---|
| Unit | Vitest — API handlers, scoring logic | — |
| Integration | API tests — DeepSpeci ↔ Requirement Evaluator ↔ Model Provider | — |
| E2E | Playwright — full import → validate → export flow | Playwright MCP |
| Performance | k6 — 100 concurrent validations | Performance Agent |
| Security | SAST/DAST on DeepSpeci API | Security Agent |

**Traceability:** Each test case maps to an FR-DS-xx ID.

---

## 10. Risks & Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| LLM hallucination in gap analysis | False gaps clutter review | Confidence threshold (< 0.65 → flag) |
| JIRA API rate limits | Import delays | Batch imports; cache JIRA data |
| PII in requirements | Compliance violation | Presidio pre-processing; block LLM call if PII found |
| Model provider outage | Feature unavailable | Multi-model fallback (Gemini → Claude → OpenAI) |

---

## 11. Roadmap Hooks

| Tag | Feature | Status |
|---|---|---|
| Mar-W1 | JIRA import integration | TODO |
| Mar-W3 | Ambiguity detection v1 | TODO |
| Apr-W1 | Testability scoring | TODO |
| Apr-W2 | Requirement-traceability linking | TODO |
| May-W1 | Natural-language chat input | TODO |
| May-W3 | Confluence & DOCX import | TODO |
| Jun-W1 | Version history & diff view | TODO |
