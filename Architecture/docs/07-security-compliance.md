---
layout: default
title: "7. Security & Compliance"
nav_order: 9
description: "Identity, network controls, data security, and LLM safety measures for QI Offerings."
permalink: "/docs/security-compliance/"
---

[← Back to Architecture Hub]({{ '/' | relative_url }})
{: .fs-3 }

# 7. Security & Compliance

---

## Identity & Access

- **Microsoft Entra ID** (formerly Azure AD) for SSO and RBAC.
- OAuth 2.0 Authorization Code flow with PKCE (SPA) and Client Credentials (service-to-service).
- Roles: `Reader`, `Contributor`, `Admin`.
- Token lifetime: 1 h access, 24 h refresh.

---

## Network Controls

- Azure API Management deployed in a **VNet-integrated** subnet.
- Private Endpoints for PostgreSQL, Blob Storage, Azure AI Search, and Azure OpenAI.
- WAF v2 (OWASP 3.2 rule set) in front of the gateway.
- IP allowlist for admin endpoints.

---

## Data Security

| Control | Implementation |
|---|---|
| Encryption at rest | AES-256 (Azure-managed keys; CMK option) |
| Encryption in transit | TLS 1.3 (minimum TLS 1.2) |
| PII handling | PII detection via Presidio; redact before indexing |
| Key management | Azure Key Vault (RBAC-based access policies) |
| Secrets in code | Zero secrets in repo; injected via Key Vault CSI driver |

---

## LLM Safety

- **Azure OpenAI content filters** enabled (severity threshold: medium).
- Per-user **rate limiting** (tokens/min + requests/min).
- Full **audit log** of every prompt and completion (stored 90 days in Log Analytics).
- Prompt injection mitigation: input delimiter tokens + instruction hierarchy.

---

## Compliance Checklist

| Area | Status |
|---|---|
| Data residency (Azure region) | ✅ Enforced via Bicep |
| PII redaction | ✅ Presidio pre-processing |
| Encryption at rest / in transit | ✅ AES-256 / TLS 1.3 |
| Audit logging | ✅ 90-day retention |
| RBAC | ✅ Entra ID roles |
| Vulnerability scanning | ✅ CI/CD gate (Trivy + CodeQL) |

---

**Previous:** [← Data & Storage]({{ '/docs/data-storage/' | relative_url }}) · **Next:** [Observability & QE →]({{ '/docs/observability-qe/' | relative_url }})
