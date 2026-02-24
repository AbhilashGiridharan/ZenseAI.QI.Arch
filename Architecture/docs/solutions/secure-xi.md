---
layout: default
title: "Solution: Secure-Xi"
description: "Requirements specification for Secure-Xi — AI-driven vulnerability scanning, threat modelling & compliance."
---

[← Back to Solutions](../roles.md) · [Architecture Hub](../../index.md)
{: .fs-3 }

# Secure-Xi — Requirements Specification

---

## 1. Problem Statement & Goals

**Problem:** Security testing is typically bolted on at the end of the SDLC. Teams lack continuous visibility into vulnerabilities, and threat models are rarely maintained.

**Goals:**
- Shift-left security by integrating AI-driven scanning into CI and QI Workspace
- Auto-generate OWASP-aligned security test cases from architecture artefacts
- Maintain a living threat model updated with every code/infra change
- Surface security posture as a first-class metric in Insights360

---

## 2. In-Scope / Out-of-Scope

| In-Scope | Out-of-Scope |
|---|---|
| SAST integration (semgrep, CodeQL) | Penetration testing (manual red-team) |
| DAST scanning (OWASP ZAP) | Incident response automation |
| AI threat model generation & updates | Firewall/WAF rule management |
| Dependency vulnerability scanning (SCA) | SOC monitoring & alerting |
| Security test case generation | Compliance auditing (SOC 2, ISO 27001) |
| Security posture badge → Insights360 | |

---

## 3. Key Capabilities

| Capability | QE Work Lane | Supporting Agent(s) |
|---|---|---|
| SAST scanning integration | Vulnerability scanning & threat analysis | Security Agent |
| DAST scanning orchestration | Vulnerability scanning & threat analysis | Security Agent |
| AI threat model generation | Vulnerability scanning & threat analysis | Security Agent |
| Security test case generation | Create/optimise test libraries | Security Agent, Test Case Generator |
| Dependency SCA scanning | Vulnerability scanning & threat analysis | Security Agent |
| Posture reporting to Insights360 | Vulnerability scanning & threat analysis | Security Agent, Report Agent |

---

## 4. Actors & Interfaces

| Actor | Interface | Human-in-the-Loop |
|---|---|---|
| Security Engineer | Web UI — scans, threat model, findings | Reviews findings; triages severity |
| Technical Manager | Dashboard — security posture badge | Monitors trends |
| Developer | IDE notification / PR comment | Fixes flagged code |
| Security Agent (AI) | Internal API | Automated; findings reviewed by engineer |

---

## 5. Architecture Summary

```
Code repo / OpenAPI spec / Infra-as-Code → Secure-Xi UI → API → Security Agent → Model Provider (Claude primary)
                                                                    ├── semgrep / CodeQL (SAST)
                                                                    ├── OWASP ZAP (DAST)
                                                                    ├── Snyk / Trivy (SCA)
                                                                    ├── Threat model store (Postgres)
                                                                    └── Findings → Insights360
```

---

## 6. Functional Requirements

| ID | Requirement | Acceptance Criteria |
|---|---|---|
| FR-SX-01 | Run SAST scan on source code | Given a code repo, When scanned, Then findings with CWE IDs returned |
| FR-SX-02 | Run DAST scan on staging URL | Given a staging URL, When scanned, Then OWASP Top 10 findings returned |
| FR-SX-03 | Generate AI threat model | Given architecture docs, When analysed, Then STRIDE-based threat model produced |
| FR-SX-04 | Update threat model on change | Given a code/infra diff, When analysed, Then threat model deltas identified |
| FR-SX-05 | Generate security test cases | Given threat model, When generated, Then test cases mapped to CWE/OWASP |
| FR-SX-06 | Dependency SCA scan | Given a lockfile (package-lock.json, pom.xml), When scanned, Then CVEs listed |
| FR-SX-07 | Severity triage assistance | Given a finding, When AI analysed, Then exploitability + impact assessed |
| FR-SX-08 | Block CI on critical findings | Given critical/high findings, When CI gate evaluated, Then pipeline blocked |
| FR-SX-09 | Publish security posture to Insights360 | Given findings summary, When published, Then posture badge updated |

### Gherkin Example (FR-SX-03)

```gherkin
Feature: AI Threat Model Generation
  Scenario: Generate threat model from architecture docs
    Given the architecture document describes a REST API with JWT auth and PostgreSQL
    When the Security Agent generates a threat model
    Then a STRIDE-based model is produced with ≥ 6 threat entries
    And each entry includes: threat, category (STRIDE), affected component, severity, mitigation
    And the model is stored in the threat-model repository
```

---

## 7. Non-Functional Requirements

| NFR | Target |
|---|---|
| SAST scan latency (100 KLOC) | ≤ 120 s |
| DAST scan latency | ≤ 300 s |
| Threat model generation | ≤ 15 s |
| SCA scan (500 deps) | ≤ 30 s |
| False positive rate (SAST) | ≤ 15 % |
| Finding deduplication accuracy | ≥ 95 % |
| Findings retention | 1 year |

---

## 8. Data Requirements

| Data | Direction | Schema |
|---|---|---|
| Source code / repo URL | Input | Git clone URL or local path |
| Architecture docs | Input | Markdown / structured JSON |
| Lockfile / BOM | Input | package-lock.json, pom.xml, etc. |
| Threat model | Output | `{threats[{id, category, component, severity, mitigation}]}` |
| SAST findings | Output | `{findings[{ruleId, cweId, file, line, severity, snippet}]}` |
| DAST findings | Output | `{findings[{alertId, owaspCategory, url, risk, evidence}]}` |
| SCA findings | Output | `{vulns[{cveId, package, version, severity, fixVersion?}]}` |
| Posture badge | Output | `{critical, high, medium, low, score, trend}` |

---

## 9. Test Strategy & Coverage

| Level | Approach | Agent |
|---|---|---|
| Unit | Jest — finding deduplication, severity scoring | — |
| Integration | Secure-Xi API ↔ Security Agent ↔ semgrep/ZAP | — |
| E2E | Scan intentionally vulnerable app (DVWA) → validate findings | Security Agent |
| Performance | k6 — concurrent scan requests | Performance Agent |
| Meta-security | SAST/DAST on Secure-Xi itself | Security Agent |

---

## 10. Risks & Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| High false positive rate | Alert fatigue | AI triage + severity confidence scoring |
| DAST scan triggers WAF blocks | Incomplete scan | Allowlist scanner IPs in staging WAF |
| Threat model drift | Stale model | Auto-trigger model update on PR merge |
| Sensitive code exposed to LLM | Data leak | Strip secrets; use on-prem model or guardrails |

---

## 11. Roadmap Hooks

| Tag | Feature | Status |
|---|---|---|
| May-W2 | SAST integration (semgrep) | TODO |
| Jun-W1 | DAST integration (ZAP) | TODO |
| Jun-W2 | AI threat model generation | TODO |
| Jun-W4 | SCA scanning | TODO |
| Jul-W1 | Security test case generation | TODO |
| Jul-W3 | CI gate + Insights360 posture badge | TODO |
