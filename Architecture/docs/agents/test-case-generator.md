---
layout: default
title: "Agent: Test Case Generator"
description: "Design spec for the Test Case Generator AI Agent."
---

[← Back to Agents](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# Test Case Generator — Agent Design Spec

---

## 1. Mission

Generate comprehensive, structured test cases from validated requirements — covering positive, negative, boundary, and edge scenarios — ready for manual review or downstream automation.

**QE Work Lane:** Formulate testing approaches; draft structured scenarios; create/optimise test libraries.

---

## 2. Inputs & Outputs

| Direction | Artifact | Format |
|---|---|---|
| **Input** | Validated requirements (from Requirement Evaluator) | JSON / Markdown |
| **Input** | Existing test library (optional, for dedup) | JSON / CSV |
| **Input** | Domain rules / constraints | YAML |
| **Output** | Test cases (Gherkin + metadata) | Markdown / JSON |
| **Output** | Coverage matrix (requirement → test case mapping) | CSV / Markdown table |
| **Output** | Priority / risk scores per test case | JSON |

---

## 3. LLM / Model Usage

| Model | Use Case | Selection Logic |
|---|---|---|
| **Gemini** | Bulk generation from large requirement sets | Token count > 30 K |
| **Claude** | High-fidelity Gherkin with nuanced edge cases | Quality-critical paths |
| **OpenAI** | Fast generation, structured JSON output | Default / cost-optimised |

---

## 4. Orchestration

```
CaseGeni UI → /api/agents/test-case-generator/generate
                ├── Receive validated reqs (from DeepSpeci or direct)
                ├── Dedup against existing test library
                ├── Call LLM → generate Gherkin scenarios
                ├── Post-process: tag priority, map to requirements
                └── Return to CaseGeni → human review
```

- **Caller:** CaseGeni (primary), DeepSpeci (inline generation from requirements)
- **Consumers:** CaseGeni (test library), Auto-PlayPilot (automation input), DataGeni (data dependency extraction)

---

## 5. Guardrails

| Guardrail | Implementation |
|---|---|
| **Deduplication** | Semantic similarity check against existing library (threshold ≥ 0.92) |
| **Hallucination** | Every test step must trace to a requirement ID |
| **Completeness** | Enforce min coverage: ≥ 1 positive + 1 negative per requirement |
| **Determinism** | Temperature = 0.2; structured output with JSON schema |
| **PII** | No real user data in test case examples; flag for synthetic data from DataGeni |

---

## 6. APIs / Tools

| Tool | Purpose |
|---|---|
| CaseGeni Test Library API | Fetch existing test cases for dedup |
| Requirement Evaluator API | Pull validated requirements |
| VCS (GitHub API) | Link test cases to source artefacts |
| Schema Validator | Enforce Gherkin / JSON output schema |

---

## 7. Evaluation

| KPI | Target | Method |
|---|---|---|
| Requirement coverage | 100 % (≥ 1 test per req) | Automated coverage check |
| Edge-case detection rate | ≥ 75 % | Golden set with known edge cases |
| Duplicate rate | < 5 % | Semantic dedup audit |
| Gherkin syntax validity | 100 % | Parser validation |
| Latency (p95) | ≤ 8 s per requirement batch | End-to-end measurement |

---

## 8. Failure Modes & Fallbacks

| Failure Mode | Fallback |
|---|---|
| Model generates incomplete Gherkin | Re-prompt with schema enforcement |
| Requirements input malformed | Return validation error to CaseGeni |
| Dedup service unavailable | Generate with warning "dedup skipped" |
| Token limit exceeded | Chunk requirements, generate in batches |
| Low confidence output | Flag for human review with partial results |
