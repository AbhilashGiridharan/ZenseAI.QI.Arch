---
layout: default
title: "Agent: Test Data Generator"
description: "Design spec for the Test Data Generator AI Agent."
---

[← Back to Agents](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# Test Data Generator — Agent Design Spec

---

## 1. Mission

Generate realistic, schema-compliant synthetic test data — covering positive, negative, boundary, and edge-case conditions — while ensuring zero PII leakage and deterministic reproducibility.

**QE Work Lane:** Define data conditions & dependencies; synthetic data needs.

---

## 2. Inputs & Outputs

| Direction | Artifact | Format |
|---|---|---|
| **Input** | Test cases with data dependencies (from Test Case Generator) | JSON |
| **Input** | Database schema / data dictionary | DDL / JSON Schema |
| **Input** | Data constraints & business rules | YAML |
| **Input** | Volume / cardinality requirements | JSON config |
| **Output** | Synthetic datasets | CSV, JSON, SQL INSERT, Parquet |
| **Output** | Data lineage report (which test case uses which data) | Markdown / JSON |
| **Output** | PII compliance certificate | JSON (`{scan_result, redacted_fields}`) |

---

## 3. LLM / Model Usage

| Model | Use Case | Selection Logic |
|---|---|---|
| **Gemini** | Complex relational data generation with FK constraints | Multi-table schemas |
| **OpenAI** | Fast tabular data, structured JSON output | Default for simple schemas |
| **Claude** | Edge-case data reasoning (financial, medical domains) | Domain-critical data |

---

## 4. Orchestration

```
DataGeni UI → /api/agents/test-data-generator/generate
                ├── Parse schema + constraints
                ├── Extract data needs from test cases
                ├── Call LLM → generate synthetic records
                ├── Validate: schema compliance, FK integrity, PII scan
                └── Return to DataGeni → human review → export
```

- **Caller:** DataGeni (primary), CaseGeni (inline data for scenarios)
- **Consumers:** DataGeni (datasets), Auto-PlayPilot (test fixtures), Perf-Xi (load test data volumes)

---

## 5. Guardrails

| Guardrail | Implementation |
|---|---|
| **PII prevention** | Never use real names, SSNs, emails; Faker-style synthetic values |
| **PII scan** | Post-generation Presidio scan; block if PII detected |
| **Schema validation** | Validate every record against JSON Schema / DDL |
| **FK integrity** | Enforce referential integrity across related tables |
| **Determinism** | Seed-based generation for reproducibility |
| **Volume limits** | Cap at 10 K records per request; paginate larger sets |

---

## 6. APIs / Tools

| Tool | Purpose |
|---|---|
| Database introspection API | Fetch live schema metadata |
| Faker / Mimesis libraries | Locale-aware synthetic data primitives |
| Presidio | Post-generation PII scan |
| Schema Validator | JSON Schema / DDL compliance check |
| DataGeni export API | Push generated data to test environments |

---

## 7. Evaluation

| KPI | Target | Method |
|---|---|---|
| Schema compliance rate | 100 % | Automated schema validation |
| PII leakage rate | 0 % | Presidio scan on every output |
| FK integrity pass rate | 100 % | Cross-table relationship check |
| Data diversity score | ≥ 0.8 (0–1 scale) | Entropy analysis on generated values |
| Latency (p95) | ≤ 10 s for 1 K records | End-to-end measurement |

---

## 8. Failure Modes & Fallbacks

| Failure Mode | Fallback |
|---|---|
| PII detected in output | Block + re-generate with stricter constraints |
| Schema validation failure | Return errors with field-level fix suggestions |
| FK constraint violation | Re-generate dependent records |
| Model timeout on large volume | Batch into smaller chunks; merge results |
| Unsupported data type | Fall back to Faker library for that column |
