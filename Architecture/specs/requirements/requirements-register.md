---
layout: default
title: "Functional & Non-Functional Requirements Register"
description: "Consolidated FR/NFR register across all ZenseAI.Qi solutions."
---

[← Architecture Hub](../../index.md)
{: .fs-3 }

# Functional & Non-Functional Requirements Register

> Single source of truth for every requirement across the ZenseAI.Qi platform.

---

## How to Use

- **FR-XX-NN** — Functional Requirement. Prefix indicates solution (DS = DeepSpeci, CG = CaseGeni, DG = DataGeni, AP = Auto-PlayPilot, I3 = Insights360, PX = Perf-Xi, SX = Secure-Xi).
- **NFR** — Non-Functional Requirement linked to one or more solutions.
- Status: `DRAFT` → `APPROVED` → `IMPLEMENTED` → `VERIFIED`

---

## Functional Requirements — DeepSpeci

| ID | Requirement | Priority | Status |
|---|---|---|---|
| FR-DS-01 | Upload & parse requirements (PDF, DOCX, Confluence, JIRA) | P0 | DRAFT |
| FR-DS-02 | AI ambiguity detection with highlighted spans | P0 | DRAFT |
| FR-DS-03 | Suggest rewrites for ambiguous requirements | P0 | DRAFT |
| FR-DS-04 | Completeness scoring (0-100) | P1 | DRAFT |
| FR-DS-05 | Testability scoring (0-100) | P1 | DRAFT |
| FR-DS-06 | Requirement versioning & diff | P1 | DRAFT |
| FR-DS-07 | Export evaluated requirements (Markdown, DOCX) | P2 | DRAFT |
| FR-DS-08 | Batch processing (≤ 50 requirements) | P1 | DRAFT |
| FR-DS-09 | Audit trail of all evaluations | P1 | DRAFT |

## Functional Requirements — CaseGeni

| ID | Requirement | Priority | Status |
|---|---|---|---|
| FR-CG-01 | Generate test cases from evaluated requirements | P0 | DRAFT |
| FR-CG-02 | Generate in Gherkin / structured tabular format | P0 | DRAFT |
| FR-CG-03 | Apply coverage techniques (BVA, EP, pairwise) | P0 | DRAFT |
| FR-CG-04 | Traceability matrix (requirement → test case) | P0 | DRAFT |
| FR-CG-05 | Edge/negative case generation | P1 | DRAFT |
| FR-CG-06 | Bulk generation (≤ 50 requirements → cases) | P1 | DRAFT |
| FR-CG-07 | Edit & approve generated cases | P1 | DRAFT |
| FR-CG-08 | Export to ALM tools (JIRA Xray, qTest) | P2 | DRAFT |
| FR-CG-09 | Coverage gap detection | P1 | DRAFT |

## Functional Requirements — DataGeni

| ID | Requirement | Priority | Status |
|---|---|---|---|
| FR-DG-01 | Generate synthetic data from schema + constraints | P0 | DRAFT |
| FR-DG-02 | Infer schema from DB connection / DDL | P0 | DRAFT |
| FR-DG-03 | PII-safe generation (no real data leakage) | P0 | DRAFT |
| FR-DG-04 | Maintain referential integrity (FK relationships) | P0 | DRAFT |
| FR-DG-05 | Volume scaling (up to 1 M rows) | P1 | DRAFT |
| FR-DG-06 | Data masking of production snapshots | P1 | DRAFT |
| FR-DG-07 | Export (CSV, JSON, SQL INSERT, Parquet) | P1 | DRAFT |
| FR-DG-08 | Seed data versioning | P2 | DRAFT |
| FR-DG-09 | Statistical distribution matching | P1 | DRAFT |

## Functional Requirements — Auto-PlayPilot

| ID | Requirement | Priority | Status |
|---|---|---|---|
| FR-AP-01 | Generate Playwright .spec.ts from test cases | P0 | DRAFT |
| FR-AP-02 | Generate Page Object Models for new pages | P0 | DRAFT |
| FR-AP-03 | Use data-testid selectors preferentially | P0 | DRAFT |
| FR-AP-04 | AI troubleshoot failing tests | P1 | DRAFT |
| FR-AP-05 | Execute tests against staging environment | P0 | DRAFT |
| FR-AP-06 | Visual regression with screenshot comparison | P2 | DRAFT |
| FR-AP-07 | Commit scripts to VCS | P1 | DRAFT |
| FR-AP-08 | Integrate test fixtures from DataGeni | P1 | DRAFT |

## Functional Requirements — Insights360

| ID | Requirement | Priority | Status |
|---|---|---|---|
| FR-I3-01 | Aggregate test results from all solutions | P0 | DRAFT |
| FR-I3-02 | Generate AI release-readiness narrative | P0 | DRAFT |
| FR-I3-03 | Display coverage heatmap | P1 | DRAFT |
| FR-I3-04 | Trend charts (7d, 30d, release) | P1 | DRAFT |
| FR-I3-05 | Drill-down from KPI to individual test | P1 | DRAFT |
| FR-I3-06 | Export dashboard as PDF/CSV | P2 | DRAFT |
| FR-I3-07 | Send daily digest notification | P2 | DRAFT |
| FR-I3-08 | Defect-to-test linkage | P1 | DRAFT |
| FR-I3-09 | Performance SLI overlay on dashboard | P1 | DRAFT |

## Functional Requirements — Perf-Xi

| ID | Requirement | Priority | Status |
|---|---|---|---|
| FR-PX-01 | Generate k6 scripts from OpenAPI spec | P0 | DRAFT |
| FR-PX-02 | Define SLIs per endpoint | P0 | DRAFT |
| FR-PX-03 | Define SLO targets per SLI | P0 | DRAFT |
| FR-PX-04 | Configure load profiles | P1 | DRAFT |
| FR-PX-05 | Execute load tests | P0 | DRAFT |
| FR-PX-06 | Detect SLO breaches | P0 | DRAFT |
| FR-PX-07 | AI regression analysis | P1 | DRAFT |
| FR-PX-08 | AI capacity-planning report | P2 | DRAFT |
| FR-PX-09 | Publish SLI data to Insights360 | P1 | DRAFT |

## Functional Requirements — Secure-Xi

| ID | Requirement | Priority | Status |
|---|---|---|---|
| FR-SX-01 | Run SAST scan on source code | P0 | DRAFT |
| FR-SX-02 | Run DAST scan on staging URL | P0 | DRAFT |
| FR-SX-03 | Generate AI threat model | P0 | DRAFT |
| FR-SX-04 | Update threat model on change | P1 | DRAFT |
| FR-SX-05 | Generate security test cases | P1 | DRAFT |
| FR-SX-06 | Dependency SCA scan | P0 | DRAFT |
| FR-SX-07 | Severity triage assistance | P1 | DRAFT |
| FR-SX-08 | Block CI on critical findings | P0 | DRAFT |
| FR-SX-09 | Publish security posture to Insights360 | P1 | DRAFT |

---

## Non-Functional Requirements — Platform-Wide

| NFR ID | Metric | Target | Applies To |
|---|---|---|---|
| NFR-01 | API response time (p95) | ≤ 2 s | All solutions |
| NFR-02 | Availability | 99.5 % | All solutions |
| NFR-03 | Concurrent users | 100 | All solutions |
| NFR-04 | Data encryption at rest | AES-256 | All solutions |
| NFR-05 | Data encryption in transit | TLS 1.3 | All solutions |
| NFR-06 | Audit log retention | 1 year | All solutions |
| NFR-07 | LLM token budget per request | ≤ 8 000 tokens | All AI agents |
| NFR-08 | Prompt injection detection | Block ≥ 95 % | All AI agents |
| NFR-09 | PII-free prompts | 100 % | All AI agents |
| NFR-10 | Deployment rollback time | ≤ 5 min | Platform |
