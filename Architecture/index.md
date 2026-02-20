---
layout: default
title: "QI Offerings – Architecture Hub"
nav_order: 1
description: "Architecture documentation hub for the QI Offerings platform."
permalink: "/"
---

<!-- Hero Image -->
<img src="{{ '/assets/images/qi-offerings-overview.png' | relative_url }}"
     alt="Solution overview – QI Offerings"
     width="100%"
     style="border-radius: 6px; margin-bottom: 1.5rem;" />

# QI Offerings – Architecture Hub

Welcome to the **QI Offerings** architecture documentation site. This hub provides comprehensive technical references for the end-to-end platform with LLM + RAG integration.

---

## 📚 Documentation Pages

| # | Document | Description |
|---|---|---|
| 📄 | [**Full Architecture (single page)**]({{ '/architecture/' | relative_url }}) | Complete E2E architecture in one document |
| 1 | [**Overview**]({{ '/docs/overview/' | relative_url }}) | Problem statement, success criteria, constraints, capability map |
| 2 | [**Architecture at a Glance**]({{ '/docs/architecture-at-a-glance/' | relative_url }}) | High-level context diagram (Mermaid) |
| 3 | [**AI Integration (LLM + RAG)**]({{ '/docs/ai-integration/' | relative_url }}) | LLM providers, embeddings, vector store, retrieval pipeline, guardrails |
| 4 | [**UI/UX**]({{ '/docs/ui-ux/' | relative_url }}) | React / Next.js stack, key screens, RAG UX patterns |
| 5 | [**Backend & APIs**]({{ '/docs/backend-apis/' | relative_url }}) | FastAPI services, API gateway, async processing, component diagram |
| 6 | [**Data & Storage**]({{ '/docs/data-storage/' | relative_url }}) | PostgreSQL, Azure AI Search, Blob Storage, ERD |
| 7 | [**Security & Compliance**]({{ '/docs/security-compliance/' | relative_url }}) | Entra ID, network controls, encryption, LLM safety |
| 8 | [**Observability & QE**]({{ '/docs/observability-qe/' | relative_url }}) | OpenTelemetry, metrics, test automation agent |
| 9 | [**Deployment & Environments**]({{ '/docs/deployment/' | relative_url }}) | AKS, CI/CD (GitHub Actions), Bicep IaC, blue/green |
| 10 | [**Non-Functional Requirements**]({{ '/docs/nfr/' | relative_url }}) | SLAs, latency targets, scalability, resilience |
| 11 | [**Appendix**]({{ '/docs/appendix/' | relative_url }}) | OpenAPI spec, prompt templates, config examples |

---

## Audience

| Persona | Start Here |
|---|---|
| **Stakeholders** | [Overview]({{ '/docs/overview/' | relative_url }}) → [NFR]({{ '/docs/nfr/' | relative_url }}) |
| **Developers** | [Backend]({{ '/docs/backend-apis/' | relative_url }}) → [AI Integration]({{ '/docs/ai-integration/' | relative_url }}) → [Appendix]({{ '/docs/appendix/' | relative_url }}) |
| **QE Engineers** | [Observability & QE]({{ '/docs/observability-qe/' | relative_url }}) → [Deployment]({{ '/docs/deployment/' | relative_url }}) |
| **Security Engineers** | [Security]({{ '/docs/security-compliance/' | relative_url }}) → [Data]({{ '/docs/data-storage/' | relative_url }}) |

---

## Tech Stack Overview

```
┌─────────────┐   ┌──────────────────┐   ┌────────────────────┐
│  React /    │   │  Azure API       │   │  FastAPI Services  │
│  Next.js 15 │──▶│  Management      │──▶│  (Python 3.12)     │
└─────────────┘   └──────────────────┘   └────────┬───────────┘
                                                   │
                    ┌──────────────────────────────┼──────────────┐
                    │                              │              │
              ┌─────▼──────┐  ┌────────────┐  ┌───▼──────┐  ┌───▼───────┐
              │ Azure AI   │  │ Azure      │  │ Postgres │  │ Azure     │
              │ Search     │  │ OpenAI     │  │ 16       │  │ Blob      │
              │ (Vectors)  │  │ (GPT-4.1)  │  │          │  │ Storage   │
              └────────────┘  └────────────┘  └──────────┘  └───────────┘
```

---

<p style="text-align:center; color:#888; font-size:0.85rem;">
QI Offerings · Architecture Docs · 2026
</p>
