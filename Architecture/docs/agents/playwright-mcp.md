---
layout: default
title: "Agent: Playwright MCP"
description: "Design spec for the Playwright MCP AI Agent."
---

[← Back to Agents](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# Playwright MCP — Agent Design Spec

---

## 1. Mission

Convert structured test cases into executable Playwright automation scripts, manage test execution orchestration, and troubleshoot failing scripts using AI-assisted debugging.

**QE Work Lane:** Create / optimise test libraries; AI-assisted automation workflows; troubleshoot failing scripts.

---

## 2. Inputs & Outputs

| Direction | Artifact | Format |
|---|---|---|
| **Input** | Test cases (Gherkin / structured) from Test Case Generator | JSON / Markdown |
| **Input** | Application URL / selector map / page objects | JSON config |
| **Input** | Test data fixtures (from Test Data Generator) | JSON / CSV |
| **Input** | Failing test logs (for troubleshooting) | Text / JSON |
| **Output** | Playwright test scripts (TypeScript) | `.spec.ts` files |
| **Output** | Page Object Models | `.ts` files |
| **Output** | Execution report | JSON / HTML |
| **Output** | Fix suggestions for failing scripts | Markdown |

---

## 3. LLM / Model Usage

| Model | Use Case | Selection Logic |
|---|---|---|
| **OpenAI** | Code generation (Playwright TypeScript) | Primary — strong code-gen |
| **Claude** | Complex test logic, multi-step flows | Long scenarios > 20 steps |
| **Gemini** | Selector inference from page HTML | Multi-modal (screenshot analysis) |

---

## 4. Orchestration

```
Auto-PlayPilot UI → /api/agents/playwright-mcp/generate
                      ├── Receive test cases + app config
                      ├── Generate Page Objects (if new pages)
                      ├── Call LLM → generate .spec.ts scripts
                      ├── Lint & syntax validate
                      └── Return to Auto-PlayPilot → human review → execute

Auto-PlayPilot UI → /api/agents/playwright-mcp/troubleshoot
                      ├── Receive failing test log + script
                      ├── Call LLM → diagnose root cause
                      ├── Suggest fix (code patch)
                      └── Return to Auto-PlayPilot → engineer review
```

- **Caller:** Auto-PlayPilot (primary)
- **Consumers:** Auto-PlayPilot (scripts), Insights360 (execution results), Perf-Xi (performance-tagged tests)

---

## 5. Guardrails

| Guardrail | Implementation |
|---|---|
| **Code safety** | No `eval()`, no external network calls in generated scripts |
| **Selector stability** | Prefer `data-testid` > `aria-*` > CSS > XPath |
| **Determinism** | Temperature = 0.0 for code gen; fixed seed for data fixtures |
| **Lint enforcement** | ESLint + Playwright test linter on all generated code |
| **Secret handling** | No hardcoded credentials; use env variables |
| **Execution sandbox** | Scripts run in isolated containers; no host access |

---

## 6. APIs / Tools

| Tool | Purpose |
|---|---|
| Playwright Test Runner | Execute generated scripts |
| ESLint + TypeScript compiler | Validate generated code |
| GitHub API (VCS) | Commit scripts to test repo |
| Auto-PlayPilot execution API | Trigger test runs, collect results |
| Screenshot diff service | Visual regression comparison |

---

## 7. Evaluation

| KPI | Target | Method |
|---|---|---|
| Script compilation rate | 100 % | TypeScript compile check |
| First-run pass rate | ≥ 70 % | Execute generated scripts against staging |
| Selector stability (30 d) | ≥ 95 % | Track selector breakage over time |
| Troubleshoot fix accuracy | ≥ 60 % | Compare AI fix to engineer fix |
| Code generation latency (p95) | ≤ 12 s per test case | End-to-end measurement |

---

## 8. Failure Modes & Fallbacks

| Failure Mode | Fallback |
|---|---|
| Generated script fails to compile | Re-generate with stricter schema; show lint errors |
| Selector not found on page | Fall back to AI selector inference (Gemini vision) |
| Test execution timeout | Retry with increased timeout; flag for human review |
| Model generates unsafe code | Blocked by lint rules; logged for audit |
| Troubleshoot diagnosis incorrect | Escalate to Automation Engineer with full context |
