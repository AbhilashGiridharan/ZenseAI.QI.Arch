---
layout: default
title: "Roles & Personas"
nav_order: 3
description: "Stakeholders, business users, and skilled QI users within ZenseAI.Qi."
---

[← Back to Architecture Hub](../index.md)
{: .fs-3 }

# Roles & Personas

---

## Stakeholders / Business Users

| Role | Description | Primary Solutions | Interaction Mode |
|---|---|---|---|
| **Manager** | Oversees QE outcomes, tracks progress, approves releases | DeepSpeci, Insights360 | Dashboards, reports, requirement sign-off |
| **Developer** | Writes application code, reviews test results, fixes defects | DeepSpeci, CaseGeni, Auto-PlayPilot | Code-level integration, CI/CD feedback |
| **Non-Technical User** | Business analyst, product owner — defines requirements in natural language | DeepSpeci, Insights360 | Conversational UI, natural-language queries |
| **SME (Subject-Matter Expert)** | Domain expert who validates AI-generated outputs for accuracy | DeepSpeci, CaseGeni, DataGeni | Review & approve workflows, golden-set curation |

---

## Skilled QI Users

| Role | QE Work Lane | Primary Solutions | Primary Agents |
|---|---|---|---|
| **Technical Manager** | Requirements interpretation; testing strategy | DeepSpeci, CaseGeni, Insights360 | Requirement Evaluator, Report Agent |
| **Automation Engineer** | Test library creation; AI-assisted automation; troubleshooting | CaseGeni, DataGeni, Auto-PlayPilot | Test Case Generator, Test Data Generator, Playwright MCP |
| **Security Engineer** | Vulnerability scanning; anomalous behaviour detection | Secure-Xi | Security Agent |
| **Performance Engineer** | Performance test coordination; observability & analysis | Perf-Xi, Insights360 | Performance Agent, Report Agent |

---

## RACI Matrix

| Activity | Manager | Developer | Non-Tech User | SME | Tech Manager | Automation Eng. | Security Eng. | Performance Eng. |
|---|---|---|---|---|---|---|---|---|
| Define requirements | I | C | R | C | A | I | I | I |
| Validate requirements (AI) | I | I | I | A | R | I | I | I |
| Generate test cases | I | C | I | C | A | R | I | I |
| Generate test data | I | I | I | C | I | R | I | I |
| Create automation scripts | I | C | I | I | I | R | I | I |
| Execute performance tests | I | I | I | I | I | I | I | R |
| Run security scans | I | I | I | I | I | I | R | I |
| Review reports & dashboards | R | C | R | C | A | I | I | I |
| Approve release | A | I | C | C | R | I | I | I |

> **R** = Responsible · **A** = Accountable · **C** = Consulted · **I** = Informed

---

## Human-in-the-Loop Moments

All AI Agent outputs pass through human review before becoming authoritative:

1. **Requirement Evaluator** → Technical Manager / SME approves validated requirements
2. **Test Case Generator** → Automation Engineer / Tech Manager reviews generated scenarios
3. **Test Data Generator** → Automation Engineer confirms data adequacy and PII compliance
4. **Playwright MCP** → Automation Engineer reviews and runs generated scripts
5. **Report Agent** → Manager / Tech Manager validates report accuracy
6. **Performance Agent** → Performance Engineer interprets metrics and approves baselines
7. **Security Agent** → Security Engineer triages findings and confirms severity

---

**Previous:** [← Platform Architecture](architecture.md) · **Next:** [Agents →](agents/)
