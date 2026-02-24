---
layout: default
title: "Agent: Requirement Evaluator"
description: "Design spec for the Requirement Evaluator AI Agent."
---

[← Back to Agents](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# Requirement Evaluator — Agent Design Spec

---

## 1. Mission

Interpret, validate, and enrich business requirements to ensure they are complete, unambiguous, testable, and traceable before downstream QE activities begin.

**QE Work Lane:** Requirements interpretation / validation; formulate testing approaches; draft structured scenarios.

---

## 2. Inputs & Outputs

| Direction | Artifact | Format |
|---|---|---|
| **Input** | Raw requirements (user stories, PRDs, BRDs) | Markdown, DOCX, plain text, JIRA JSON |
| **Input** | Domain glossary / taxonomy | JSON / YAML |
| **Input** | Historical requirement quality data | JSON (optional) |
| **Output** | Validated requirement set with gap analysis | Structured Markdown / JSON |
| **Output** | Ambiguity report (flagged items + suggestions) | Markdown table |
| **Output** | Testability score per requirement | JSON (`{reqId, score, reason}`) |
| **Output** | Requirement traceability matrix (initial) | CSV / Markdown table |

---

## 3. LLM / Model Usage

| Model | Use Case | Selection Logic |
|---|---|---|
| **Gemini** | Large-context requirement analysis (long BRDs) | Default for documents > 50 K tokens |
| **Claude** | Nuanced ambiguity detection, reasoning | Preferred for quality-critical validation |
| **OpenAI** | General analysis, fallback | Cost-optimised fallback; tool-use tasks |

**Selection logic:** Model router picks based on (1) input token count, (2) task type (analysis vs. structuring), (3) cost budget for the tenant.

---

## 4. Orchestration

```
DeepSpeci UI → /api/agents/requirement-evaluator/validate
                  ├── Pre-process: sanitise, chunk if needed
                  ├── Call LLM (selected model)
                  ├── Post-process: structure output, score
                  └── Return to DeepSpeci → human review
```

- **Caller:** DeepSpeci (primary), CaseGeni (secondary — to validate before test generation)
- **Consumers:** DeepSpeci (validated reqs), CaseGeni (testable reqs feed), Insights360 (quality metrics)

---

## 5. Guardrails

| Guardrail | Implementation |
|---|---|
| **PII handling** | Scan inputs with Presidio; redact PII before LLM call; restore on output |
| **Hallucination check** | Every output claim must reference a source requirement ID |
| **Confidence threshold** | Validation confidence < 0.65 triggers human-review flag |
| **Determinism** | Temperature = 0.1 for validation; structured output schema enforced |
| **Token budget** | Max 8 K completion tokens per request |

---

## 6. APIs / Tools

| Tool | Purpose |
|---|---|
| JIRA REST API | Pull requirements from backlog |
| Confluence API | Fetch linked BRDs / PRDs |
| VCS (GitHub API) | Link requirements to code artefacts |
| Presidio | PII detection & redaction |
| Schema Validator | Enforce output JSON schema |

---

## 7. Evaluation

| KPI | Target | Method |
|---|---|---|
| Gap detection recall | ≥ 90 % | Golden set of 100 requirements with known gaps |
| Ambiguity precision | ≥ 85 % | Human-labelled ambiguity dataset |
| Testability scoring accuracy | ± 0.1 vs. human score | Weekly regression on 50 scored requirements |
| Latency (p95) | ≤ 5 s | End-to-end including LLM call |
| User acceptance rate | ≥ 80 % | Tracking "accepted" vs. "revised" in DeepSpeci |

---

## 8. Failure Modes & Fallbacks

| Failure Mode | Fallback |
|---|---|
| Primary model timeout | Retry once → switch to OpenAI (fallback) |
| Input too large (> context window) | Chunk with overlap → merge outputs |
| PII detected in output | Block response; re-run with stricter redaction |
| Confidence below threshold | Return partial results with human-review banner |
| Upstream API (JIRA) down | Queue request; notify user of delay |
