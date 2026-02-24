---
layout: default
title: "Agent: Security Agent"
description: "Design spec for the Security Agent AI Agent."
---

[← Back to Agents](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# Security Agent — Agent Design Spec

---

## 1. Mission

Identify vulnerabilities through AI-powered scanning, detect anomalous behaviour with intelligent threat analysis, and provide prioritised remediation guidance — integrated into the QE workflow.

**QE Work Lane:** Identify vulnerabilities with AI-powered scanning; detect anomalous behaviour with intelligent threat analysis.

---

## 2. Inputs & Outputs

| Direction | Artifact | Format |
|---|---|---|
| **Input** | Application endpoints / API specs | OpenAPI YAML |
| **Input** | Source code / repository URL | Git URL |
| **Input** | Infrastructure config (Bicep / Terraform / K8s manifests) | IaC files |
| **Input** | Runtime logs and telemetry | JSON / log streams |
| **Input** | OWASP / CWE rule sets | YAML config |
| **Output** | Vulnerability findings | SARIF / JSON |
| **Output** | Threat model (STRIDE-based) | Markdown |
| **Output** | Remediation recommendations (prioritised) | Markdown |
| **Output** | Anomaly detection alerts | JSON (webhook-compatible) |
| **Output** | Compliance posture report | Markdown / PDF |

---

## 3. LLM / Model Usage

| Model | Use Case | Selection Logic |
|---|---|---|
| **OpenAI** | Code analysis, SAST-style vulnerability detection | Primary — tool-use + code understanding |
| **Claude** | Threat modelling narrative, remediation reasoning | Complex security analysis |
| **Gemini** | Large codebase scanning (long context) | Repos > 100 K tokens |

---

## 4. Orchestration

```
Secure-Xi UI → /api/agents/security-agent/scan
                 ├── Fetch target (API spec / code / infra)
                 ├── Run rule-based scanners (Trivy, CodeQL, ZAP)
                 ├── Call LLM → enrich findings, prioritise, suggest fixes
                 └── Return to Secure-Xi → Security Engineer review

Secure-Xi UI → /api/agents/security-agent/threat-model
                 ├── Analyse architecture + data flows
                 ├── Call LLM → STRIDE threat model
                 └── Return to Secure-Xi → Security Engineer review

Continuous:
  Runtime telemetry → /api/agents/security-agent/anomaly-detect
                        ├── Stream analysis (sliding window)
                        ├── ML anomaly detection + LLM contextualisation
                        └── Alert if anomaly confirmed → Secure-Xi + Insights360
```

- **Caller:** Secure-Xi (primary), CI/CD pipeline (pre-deploy gate)
- **Consumers:** Secure-Xi (findings dashboard), Insights360 (security posture), CI/CD (security gate)

---

## 5. Guardrails

| Guardrail | Implementation |
|---|---|
| **False positive rate** | Confidence scoring; threshold ≥ 0.7 for auto-report |
| **Sensitive data** | Never log secrets, tokens, or credentials found during scanning |
| **Scope control** | Only scan authorised repositories and endpoints |
| **Rate limiting** | Max 3 concurrent scans per tenant |
| **Audit trail** | Full log of every scan, finding, and remediation action |
| **Isolation** | Scans run in sandboxed containers; no write access to target |

---

## 6. APIs / Tools

| Tool | Purpose |
|---|---|
| Trivy | Container & dependency vulnerability scanning |
| CodeQL | Static application security testing (SAST) |
| OWASP ZAP | Dynamic application security testing (DAST) |
| GitHub Advanced Security | Secret scanning, Dependabot alerts |
| Presidio | PII detection in findings output |
| Snyk API | Open-source dependency analysis |

---

## 7. Evaluation

| KPI | Target | Method |
|---|---|---|
| Vulnerability detection recall | ≥ 90 % | Golden set of known vulnerabilities |
| False positive rate | < 15 % | Human review sample (weekly) |
| Remediation acceptance rate | ≥ 65 % | Track fixes applied by engineers |
| MTTR (mean time to remediate) | Reduce by 30 % vs. manual | Compare pre/post adoption |
| Anomaly detection precision | ≥ 80 % | Labelled anomaly dataset |
| Scan latency (p95) | ≤ 60 s for typical repo | End-to-end measurement |

---

## 8. Failure Modes & Fallbacks

| Failure Mode | Fallback |
|---|---|
| Scanner tool failure (Trivy/ZAP) | Retry once; skip tool but report partial scan |
| LLM enrichment timeout | Return raw scanner findings without AI prioritisation |
| False positive spike | Auto-adjust confidence threshold; alert Security Engineer |
| Credential found in scan | Immediately redact from output; trigger secret rotation alert |
| Anomaly detector overwhelmed | Sample telemetry at reduced rate; backfill analysis async |
