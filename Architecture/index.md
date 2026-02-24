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
| **Stakeholders / Managers** | [Overview](docs/01-overview.md) → [NFR](docs/10-nfr.md) → [Roles & Personas](docs/roles.md) → [Insights360](docs/solutions/insights360.md) |
| **Developers** | [Backend](docs/05-backend-apis.md) → [AI Integration](docs/03-ai-integration.md) → [Appendix](docs/11-appendix.md) |
| **QE Engineers** | [Observability & QE](docs/08-observability-qe.md) → [Test Strategy](specs/tests/test-strategy.md) → [CaseGeni](docs/solutions/casegeni.md) |
| **Automation Engineers** | [Auto-PlayPilot](docs/solutions/auto-playpilot.md) → [Playwright MCP Agent](docs/agents/playwright-mcp.md) → [DataGeni](docs/solutions/datageni.md) |
| **Security Engineers** | [Security Spec](specs/security/security-spec.md) → [Secure-Xi](docs/solutions/secure-xi.md) → [Security Agent](docs/agents/security-agent.md) |
| **Performance Engineers** | [Performance Spec](specs/performance/performance-spec.md) → [Perf-Xi](docs/solutions/perf-xi.md) → [Performance Agent](docs/agents/performance-agent.md) |
| **SMEs / Non-Technical** | [Roles & Personas](docs/roles.md) → [DeepSpeci](docs/solutions/deepspeci.md) → [Roadmap](docs/roadmap/roadmap.md) |

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

## 🚀 ZenseAI.Qi Platform

The **ZenseAI.Qi** platform builds on the QI Offerings architecture above, delivering 7 AI-powered solutions, 7 specialised agents, and an integrated QE workspace.

<!-- ZenseAI.Qi Platform Architecture -->
<img src="assets/images/zenseai-qi-platform.svg" alt="ZenseAI.Qi Platform Architecture" width="100%" style="border-radius:6px; margin-bottom:1.5rem;" />

### Architecture & Roles

| Document | Description |
|---|---|
| [**Platform Architecture**](docs/architecture.md) | Mermaid architecture diagram + solution-agent-provider mapping |
| [**Roles & Personas**](docs/roles.md) | Stakeholders, skilled users, RACI matrix, human-in-the-loop |

### Solution Architectures

| # | Solution | Architecture | Agent | Model |
|---|---|---|---|---|
| 1 | DeepSpeci | [Architecture](docs/architecture.md#1-deepspeci-architecture) | Requirement Evaluator | Gemini |
| 2 | CaseGeni | [Architecture](docs/architecture.md#2-casegeni-architecture) | Test Case Generator | Gemini |
| 3 | DataGeni | [Architecture](docs/architecture.md#3-datageni-architecture) | Test Data Generator | Claude |
| 4 | Auto-PlayPilot | [Architecture](docs/architecture.md#4-auto-playpilot-architecture) | Playwright MCP | OpenAI |
| 5 | Insights360 | [Architecture](docs/architecture.md#5-insights360-architecture) | Report Agent | Gemini |
| 6 | Perf-Xi | [Architecture](docs/architecture.md#6-perf-xi-architecture) | Performance Agent | Claude |
| 7 | Secure-Xi | [Architecture](docs/architecture.md#7-secure-xi-architecture) | Security Agent | Claude |

### AI Agent Design Specs

| Agent | Primary Solution | Document |
|---|---|---|
| Requirement Evaluator | DeepSpeci | [Design Spec](docs/agents/requirement-evaluator.md) |
| Test Case Generator | CaseGeni | [Design Spec](docs/agents/test-case-generator.md) |
| Test Data Generator | DataGeni | [Design Spec](docs/agents/test-data-generator.md) |
| Playwright MCP | Auto-PlayPilot | [Design Spec](docs/agents/playwright-mcp.md) |
| Report Agent | Insights360 | [Design Spec](docs/agents/report-agent.md) |
| Performance Agent | Perf-Xi | [Design Spec](docs/agents/performance-agent.md) |
| Security Agent | Secure-Xi | [Design Spec](docs/agents/security-agent.md) |

### Solution Requirement Specs

| Solution | QE Work Lane | Document |
|---|---|---|
| DeepSpeci | Requirements interpretation & validation | [Requirements](docs/solutions/deepspeci.md) |
| CaseGeni | AI-assisted test automation workflows | [Requirements](docs/solutions/casegeni.md) |
| DataGeni | AI-assisted test automation workflows | [Requirements](docs/solutions/datageni.md) |
| Auto-PlayPilot | Create/optimise test libraries | [Requirements](docs/solutions/auto-playpilot.md) |
| Insights360 | Consolidated reporting & analytics | [Requirements](docs/solutions/insights360.md) |
| Perf-Xi | Performance test coordination & observability | [Requirements](docs/solutions/perf-xi.md) |
| Secure-Xi | Vulnerability scanning & threat analysis | [Requirements](docs/solutions/secure-xi.md) |

### Specifications

| Spec | Description | Document |
|---|---|---|
| Requirements Register | Consolidated FR/NFR across all solutions | [Register](specs/requirements/requirements-register.md) |
| Test Strategy | Test pyramid, traceability, coverage targets | [Strategy](specs/tests/test-strategy.md) |
| Security | Threat model, auth, data protection, guardrails | [Security](specs/security/security-spec.md) |
| Performance | SLIs/SLOs, budgets, capacity planning, runbooks | [Performance](specs/performance/performance-spec.md) |

### Roadmap

| Document | Description |
|---|---|
| [**Product Roadmap**](docs/roadmap/roadmap.md) | 5-phase delivery plan (Mar → Jul) with weekly milestones |

<!-- Agent ↔ Solution ↔ Model Provider Mapping -->
<img src="assets/images/zenseai-qi-agent-mapping.svg" alt="Agent ↔ Solution ↔ Model Provider Mapping" width="100%" style="border-radius:6px; margin:1.5rem 0;" />

---

## 🗂️ Architecture Diagrams Gallery

| Diagram | Preview |
|---|---|
| [Platform Overview](assets/images/01-platform-overview.svg) | QI Offerings end-to-end architecture |
| [AI & RAG Pipeline](assets/images/02-ai-rag-pipeline.svg) | LLM providers, embeddings, vector store |
| [UI/UX Architecture](assets/images/03-ui-ux-architecture.svg) | React / Next.js front-end layers |
| [Backend Services](assets/images/04-backend-services.svg) | FastAPI microservices component view |
| [Data & Storage](assets/images/05-data-storage.svg) | PostgreSQL, blob, vector store ERD |
| [Security Architecture](assets/images/06-security-architecture.svg) | Defence-in-depth layers |
| [Observability & QE](assets/images/07-observability-qe.svg) | OpenTelemetry, metrics, tracing |
| [Deployment Topology](assets/images/08-deployment-topology.svg) | AKS, CI/CD, blue/green |
| [ZenseAI.Qi Platform](assets/images/zenseai-qi-platform.svg) | Solutions, agents, providers, data layer |
| [Agent ↔ Solution Mapping](assets/images/zenseai-qi-agent-mapping.svg) | 7 agents mapped to solutions & models |
| [Roadmap Timeline](assets/images/zenseai-qi-roadmap.svg) | 5-phase delivery Mar → Jul |
| [DeepSpeci Architecture](assets/images/arch-deepspeci.svg) | Requirement ingestion, evaluation, validation |
| [CaseGeni Architecture](assets/images/arch-casegeni.svg) | Test case generation, dedup, coverage matrix |
| [DataGeni Architecture](assets/images/arch-datageni.svg) | Synthetic data, PII scan, FK integrity |
| [Auto-PlayPilot Architecture](assets/images/arch-auto-playpilot.svg) | Playwright codegen, POM, troubleshoot |
| [Insights360 Architecture](assets/images/arch-insights360.svg) | Aggregation, AI narrative, dashboards |
| [Perf-Xi Architecture](assets/images/arch-perf-xi.svg) | k6 generation, SLI/SLO, capacity planning |
| [Secure-Xi Architecture](assets/images/arch-secure-xi.svg) | SAST/DAST/SCA, threat model, CI gate |

---

<p style="text-align:center; color:#888; font-size:0.85rem;">
QI Offerings · ZenseAI.Qi · Architecture Docs · 2026
</p>
