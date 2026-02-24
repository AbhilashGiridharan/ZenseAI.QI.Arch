---
layout: default
title: "Security Specification"
description: "Security architecture, threat model, and compliance policies for ZenseAI.Qi platform."
---

[← Architecture Hub](../../index.md)
{: .fs-3 }

# Security Specification

---

## 1. Security Architecture Overview

ZenseAI.Qi follows a **defence-in-depth** strategy spanning network, application, data, and AI layers.

```
┌──────────────────────────────────────────────────┐
│                   WAF / CDN                      │
├──────────────────────────────────────────────────┤
│            API Gateway (rate-limit, auth)         │
├──────────────────────────────────────────────────┤
│   ┌────────────┐  ┌────────────┐  ┌───────────┐ │
│   │ Solutions   │  │ AI Agents  │  │ Admin UI  │ │
│   └─────┬──────┘  └─────┬──────┘  └─────┬─────┘ │
│         │               │               │       │
│   ┌─────▼───────────────▼───────────────▼─────┐ │
│   │        Data Layer (encrypted at rest)      │ │
│   └───────────────────────────────────────────┘ │
└──────────────────────────────────────────────────┘
```

---

## 2. Authentication & Authorisation

| Mechanism | Details |
|---|---|
| Identity Provider | Azure AD / Okta (OIDC) |
| Token format | JWT (RS256, 1 h expiry) |
| Refresh tokens | Rotating, 7-day max lifetime |
| RBAC roles | `admin`, `manager`, `engineer`, `viewer` |
| API-key auth | Service-to-service (HMAC-SHA256, rotated monthly) |
| MFA | Required for `admin` and `manager` roles |

### Permission Matrix

| Resource | admin | manager | engineer | viewer |
|---|---|---|---|---|
| Create/edit requirements | ✅ | ✅ | ✅ | ❌ |
| Run AI agents | ✅ | ✅ | ✅ | ❌ |
| View dashboards | ✅ | ✅ | ✅ | ✅ |
| Manage SLOs/SLIs | ✅ | ✅ | ✅ | ❌ |
| Security scan (trigger) | ✅ | ❌ | ✅ | ❌ |
| User management | ✅ | ❌ | ❌ | ❌ |
| Audit log access | ✅ | ✅ | ❌ | ❌ |

---

## 3. Data Protection

### Encryption

| Layer | Standard |
|---|---|
| In transit | TLS 1.3 (min TLS 1.2) |
| At rest (DB) | AES-256 (AWS RDS / Azure SQL TDE) |
| At rest (files) | AES-256 (S3 SSE-KMS / Azure Blob) |
| Secrets | HashiCorp Vault / AWS Secrets Manager |

### PII Handling

| Rule | Enforcement |
|---|---|
| No PII in LLM prompts | Regex + NER pre-scan strips PII before prompt assembly |
| No PII in logs | Structured logging with PII-field redaction |
| Synthetic data only in non-prod | DataGeni enforces PII-free generation |
| GDPR right-to-erasure | Soft-delete → 30-day hard purge |

---

## 4. AI / LLM Security

| Threat | Mitigation |
|---|---|
| Prompt injection | Input sanitisation + system-prompt anchoring + injection classifier |
| Data exfiltration via prompt | PII pre-scan; outbound token budget cap |
| Model denial-of-service | Per-user token rate limit (100 K tokens/hr) |
| Hallucinated security advice | Confidence scoring; human review gate for Secure-Xi findings |
| Supply-chain (model poisoning) | Pin model versions; validate checksums |

### Guardrail Pipeline

```
User Input → PII Scanner → Injection Classifier → Token Budget Check → LLM → Output Validator → Response
                  ↓                  ↓                    ↓                          ↓
              BLOCK if PII      BLOCK if injection    REJECT if > 8K         REDACT if PII detected
```

---

## 5. STRIDE Threat Model (Platform-Level)

| ID | Category | Threat | Component | Severity | Mitigation |
|---|---|---|---|---|---|
| TM-01 | Spoofing | Stolen JWT used by attacker | API Gateway | High | Short expiry + refresh rotation + MFA |
| TM-02 | Tampering | Modified requirements in transit | DeepSpeci API | Medium | TLS + request signing |
| TM-03 | Repudiation | User denies triggering scan | Secure-Xi | Medium | Immutable audit log |
| TM-04 | Information Disclosure | PII leaked to LLM | All agents | High | PII pre-scan guardrail |
| TM-05 | Denial of Service | Token-bombing LLM API | All agents | Medium | Per-user rate limit |
| TM-06 | Elevation of Privilege | Engineer escalates to admin | Auth service | High | RBAC + principle of least privilege |

---

## 6. Dependency & Supply-Chain Security

| Control | Tool | Frequency |
|---|---|---|
| SCA scanning | Snyk / Trivy | Every PR + weekly full scan |
| License compliance | FOSSA | Every PR |
| Container image scanning | Trivy | Every image build |
| SBOM generation | Syft | Per release |
| Critical CVE SLA | — | Patch within 48 hours |

---

## 7. Logging & Monitoring

| Signal | Destination | Retention |
|---|---|---|
| Application logs | ELK / CloudWatch | 90 days |
| Audit logs (user actions) | Immutable S3 bucket | 1 year |
| Security events | SIEM (Splunk / Sentinel) | 1 year |
| LLM prompt/response logs | Encrypted bucket (redacted) | 30 days |

### Alerting

| Event | Severity | Channel |
|---|---|---|
| Failed auth ≥ 5 in 1 min (same IP) | High | PagerDuty |
| Prompt injection detected | Medium | Slack #security |
| Critical CVE in dependency | High | PagerDuty + JIRA |
| SLO breach (security scan latency) | Medium | Slack #platform |

---

## 8. Compliance Alignment

| Standard | Relevant Controls | Status |
|---|---|---|
| OWASP Top 10 | Addressed via SAST + DAST | In progress |
| OWASP LLM Top 10 | Addressed via AI guardrails | In progress |
| GDPR (data protection) | PII handling + right-to-erasure | Planned |
| SOC 2 Type II | Audit logging + access control | Future |

---

## 9. Incident Response (High-Level)

| Phase | Action | Owner |
|---|---|---|
| Detection | Automated alerts (SIEM, guardrails) | Platform team |
| Triage | Severity assessment within 30 min | Security Engineer |
| Containment | Revoke tokens, disable affected agent | Security Engineer |
| Eradication | Patch vulnerability, rotate secrets | Engineering team |
| Recovery | Restore service, verify fix | Platform team |
| Post-mortem | Blameless review within 48 h | All stakeholders |
