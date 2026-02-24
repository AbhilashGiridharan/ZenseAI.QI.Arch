---
layout: default
title: "Solution: DataGeni"
description: "Requirements specification for DataGeni — AI-powered synthetic test data generation."
---

[← Back to Solutions](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# DataGeni — Requirements Specification

---

## 1. Problem Statement & Goals

**Problem:** Test teams spend excessive time manually crafting test data, often using stale production copies that risk PII exposure. Data gaps lead to untested edge cases.

**Goals:**
- Generate schema-compliant synthetic test data with zero PII
- Cover positive, negative, boundary, and edge-case data conditions
- Reduce test data preparation time by ≥ 70 %
- Ensure referential integrity across multi-table datasets

---

## 2. In-Scope / Out-of-Scope

| In-Scope | Out-of-Scope |
|---|---|
| Synthetic data generation (CSV, JSON, SQL, Parquet) | Production data masking / anonymisation |
| Schema import (DDL, JSON Schema, API introspection) | Data warehouse ETL |
| PII-free data guarantee with compliance scan | Live database provisioning |
| Referential integrity for multi-table datasets | Data migration tools |
| Volume scaling (up to 100 K records) | Streaming / real-time data generation |
| Integration with CaseGeni (data needs) & Auto-PlayPilot (fixtures) | |

---

## 3. Key Capabilities

| Capability | QE Work Lane | Supporting Agent(s) |
|---|---|---|
| Schema-aware synthetic data generation | Data conditions & dependencies | Test Data Generator |
| Edge-case & boundary data | Data conditions & dependencies | Test Data Generator |
| PII compliance scanning | Data conditions & dependencies | Test Data Generator |
| Multi-table FK integrity | Data conditions & dependencies | Test Data Generator |
| Volume scaling for perf tests | Performance test coordination | Test Data Generator, Performance Agent |

---

## 4. Actors & Interfaces

| Actor | Interface | Human-in-the-Loop |
|---|---|---|
| Automation Engineer | Web UI — configure schema, generate, export | Reviews data adequacy |
| Performance Engineer | Web UI / API — request high-volume datasets | Specifies volume & distribution |
| Test Data Generator (Agent) | Internal API | Automated — PII scan is automated gate |

---

## 5. Architecture Summary

```
CaseGeni (data needs) → DataGeni UI → DataGeni API → Test Data Generator Agent
                                                       ├── Model Provider
                                                       ├── Faker / Mimesis libraries
                                                       ├── Presidio (PII scan)
                                                       └── Datasets → DataGeni export
                                                                    → Auto-PlayPilot (fixtures)
                                                                    → Perf-Xi (load data)
```

---

## 6. Functional Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| FR-DG-01 | Import schema from DDL, JSON Schema, or API introspection | Given a DDL file, When imported, Then all tables/columns are parsed |
| FR-DG-02 | Generate synthetic data matching schema types & constraints | Given a schema, When generated, Then every record passes schema validation |
| FR-DG-03 | Generate edge-case / boundary data | Given numeric field (0–100), Then data includes 0, 1, 99, 100, -1, 101 |
| FR-DG-04 | Enforce referential integrity (FK constraints) | Given parent-child tables, Then every FK references a valid parent PK |
| FR-DG-05 | PII compliance scan on all generated data | Given generated data, When scanned, Then zero PII fields are detected |
| FR-DG-06 | Export in CSV, JSON, SQL INSERT, Parquet | Given a dataset, When exported, Then the selected format is valid |
| FR-DG-07 | Volume scaling up to 100 K records | Given 100 K request, Then data generated within latency SLO |
| FR-DG-08 | Seed-based reproducibility | Given the same seed + schema, When regenerated, Then identical output |
| FR-DG-09 | Data lineage (which test case consumes which data) | Given generated data, Then lineage report maps data → test cases |

---

## 7. Non-Functional Requirements

| NFR | Target |
|---|---|
| Generation latency (p95) | ≤ 10 s for 1 K records; ≤ 60 s for 100 K |
| PII leakage rate | 0 % |
| Schema validation pass rate | 100 % |
| FK integrity pass rate | 100 % |
| Availability | 99.9 % |

---

## 8. Data Requirements

| Data | Direction | Schema | PII |
|---|---|---|---|
| Database schema | Input | DDL / JSON Schema | None |
| Data constraints | Input | YAML rules | None |
| Synthetic datasets | Output | Matches input schema | Zero PII (enforced) |
| Lineage report | Output | `{datasetId, testCaseIds[], columns[]}` | None |

---

## 9. Test Strategy & Coverage

| Level | Approach | Agent |
|---|---|---|
| Unit | Vitest — generator logic, FK resolver | — |
| Integration | DataGeni ↔ Test Data Generator ↔ Faker/Presidio | — |
| E2E | Playwright — import schema → generate → export flow | Playwright MCP |
| Performance | k6 — generate 100 K records; measure latency | Performance Agent |
| Security | PII scan validation; SAST on DataGeni API | Security Agent |

---

## 10. Risks & Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| PII leaks in generated data | Compliance violation | Presidio scan gate; block export if PII found |
| Complex FK chains cause generation failure | Incomplete datasets | Topological sort + retry; warn on circular FKs |
| LLM generates invalid data types | Schema violations | Post-generation schema validation gate |

---

## 11. Roadmap Hooks

| Tag | Feature | Status |
|---|---|---|
| Mar-W3 | Schema import (DDL / JSON Schema) | TODO |
| Apr-W1 | Basic synthetic data generation | TODO |
| Apr-W3 | PII compliance scanning | TODO |
| May-W1 | FK integrity enforcement | TODO |
| May-W3 | Volume scaling (100 K) | TODO |
| Jun-W1 | Edge-case / boundary data | TODO |
| Jun-W3 | Parquet export | TODO |
