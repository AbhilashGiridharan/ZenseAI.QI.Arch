---
layout: default
title: "QI Offerings – Architecture Hub"
nav_order: 1
description: "Architecture documentation hub for the QI Offerings platform."
permalink: "/"
---

<!-- Hero Image -->
<img src="assets/images/qi-offerings-overview.svg"
     alt="Solution overview – QI Offerings"
     width="100%"
     style="border-radius: 6px; margin-bottom: 1.5rem;" />

<!-- Platform Architecture Diagram -->
![QI Offerings — End-to-End Platform Architecture](assets/images/01-platform-overview.svg)

# QI Offerings – Architecture Hub

Welcome to the **QI Offerings** architecture documentation site. This hub provides comprehensive technical references for the end-to-end platform with LLM + RAG integration.

---

## 📚 Documentation Pages

| # | Document | Description |
|---|---|---|
| 📄 | [**Full Architecture (single page)**](architecture.md) | Complete E2E architecture in one document |
| 1 | [**Overview**](docs/01-overview.md) | Problem statement, success criteria, constraints, capability map |
| 2 | [**Architecture at a Glance**](docs/02-architecture-at-a-glance.md) | High-level context diagram (Mermaid) |
| 3 | [**AI Integration (LLM + RAG)**](docs/03-ai-integration.md) | LLM providers, embeddings, vector store, retrieval pipeline, guardrails |
| 4 | [**UI/UX**](docs/04-ui-ux.md) | React / Next.js stack, key screens, RAG UX patterns |
| 5 | [**Backend & APIs**](docs/05-backend-apis.md) | FastAPI services, API gateway, async processing, component diagram |
| 6 | [**Data & Storage**](docs/06-data-storage.md) | PostgreSQL, Azure AI Search, Blob Storage, ERD |
| 7 | [**Security & Compliance**](docs/07-security-compliance.md) | Entra ID, network controls, encryption, LLM safety |
| 8 | [**Observability & QE**](docs/08-observability-qe.md) | OpenTelemetry, metrics, test automation agent |
| 9 | [**Deployment & Environments**](docs/09-deployment.md) | AKS, CI/CD (GitHub Actions), Bicep IaC, blue/green |
| 10 | [**Non-Functional Requirements**](docs/10-nfr.md) | SLAs, latency targets, scalability, resilience |
| 11 | [**Appendix**](docs/11-appendix.md) | OpenAPI spec, prompt templates, config examples |

---

## Audience

| Persona | Start Here |
|---|---|
| **Stakeholders** | [Overview](docs/01-overview.md) → [NFR](docs/10-nfr.md) |
| **Developers** | [Backend](docs/05-backend-apis.md) → [AI Integration](docs/03-ai-integration.md) → [Appendix](docs/11-appendix.md) |
| **QE Engineers** | [Observability & QE](docs/08-observability-qe.md) → [Deployment](docs/09-deployment.md) |
| **Security Engineers** | [Security](docs/07-security-compliance.md) → [Data](docs/06-data-storage.md) |

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
