# ZenseAI.Qi — Architecture Documentation

> **AI-Powered Quality Intelligence Platform** — 7 solutions, 7 AI agents, 3 model providers, unified QE workspace.

[![GitHub Pages](https://img.shields.io/badge/Docs-GitHub%20Pages-blue?logo=github)](https://abhilashgiridharan.github.io/ZenseAI.QI.Arch/)
[![Jekyll](https://img.shields.io/badge/Built%20with-Jekyll-cc0000?logo=jekyll)](https://jekyllrb.com)
[![Mermaid](https://img.shields.io/badge/Diagrams-Mermaid.js-ff3670?logo=mermaid)](https://mermaid.js.org)

---

## 🏠 Start Here

| Page | Description |
|---|---|
| [**Architecture Hub** (index.md)](Architecture/index.md) | Landing page — full navigation to every document, persona-based start guides |
| [**Full Architecture (single page)**](Architecture/architecture.md) | Complete E2E architecture reference in one document |

---

## 📚 Documentation Map

### Platform-Level

| Document | Path | Description |
|---|---|---|
| [Platform Architecture](Architecture/docs/architecture.md) | `docs/architecture.md` | Mermaid E2E diagram, agent-solution-provider mapping, QE work lanes |
| [Roles & Personas](Architecture/docs/roles.md) | `docs/roles.md` | Stakeholders, skilled users, RACI matrix, human-in-the-loop |
| [Product Roadmap](Architecture/docs/roadmap/roadmap.md) | `docs/roadmap/roadmap.md` | 5-phase delivery plan (Mar → Jul) with weekly milestones |

### QI Offerings Foundation (11 docs)

| # | Document | Path | Description |
|---|---|---|---|
| 1 | [Overview](Architecture/docs/01-overview.md) | `docs/01-overview.md` | Problem statement, success criteria, constraints, capability map |
| 2 | [Architecture at a Glance](Architecture/docs/02-architecture-at-a-glance.md) | `docs/02-architecture-at-a-glance.md` | High-level context diagram (Mermaid) |
| 3 | [AI Integration (LLM + RAG)](Architecture/docs/03-ai-integration.md) | `docs/03-ai-integration.md` | LLM providers, embeddings, vector store, retrieval pipeline |
| 4 | [UI/UX](Architecture/docs/04-ui-ux.md) | `docs/04-ui-ux.md` | React / Next.js 15 stack, key screens, RAG UX patterns |
| 5 | [Backend & APIs](Architecture/docs/05-backend-apis.md) | `docs/05-backend-apis.md` | FastAPI services, API gateway, async processing |
| 6 | [Data & Storage](Architecture/docs/06-data-storage.md) | `docs/06-data-storage.md` | PostgreSQL, Azure AI Search, Blob Storage, ERD |
| 7 | [Security & Compliance](Architecture/docs/07-security-compliance.md) | `docs/07-security-compliance.md` | Entra ID, network controls, encryption, LLM safety |
| 8 | [Observability & QE](Architecture/docs/08-observability-qe.md) | `docs/08-observability-qe.md` | OpenTelemetry, metrics, tracing, test automation agent |
| 9 | [Deployment](Architecture/docs/09-deployment.md) | `docs/09-deployment.md` | AKS, CI/CD (GitHub Actions), Bicep IaC, blue/green |
| 10 | [Non-Functional Requirements](Architecture/docs/10-nfr.md) | `docs/10-nfr.md` | SLAs, latency targets, scalability, resilience |
| 11 | [Appendix](Architecture/docs/11-appendix.md) | `docs/11-appendix.md` | OpenAPI spec, prompt templates, config examples |

---

## 🚀 Solution Architectures (7 solutions)

Each solution has a **dedicated architecture & implementation** document covering: Architecture Overview (SVG + Mermaid), Component Breakdown, Tech Stack, API Contracts, Data Model (ER diagram), Integration Patterns, Deployment Config (K8s), Folder Structure, Security, Performance Targets, and Implementation Roadmap.

| # | Solution | Architecture & Implementation | Requirements Spec | Agent Design |
|---|---|---|---|---|
| 1 | **DeepSpeci** | [📐 Architecture](Architecture/docs/architecture/deepspeci.md) | [📄 Requirements](Architecture/docs/solutions/deepspeci.md) | [🤖 Requirement Evaluator](Architecture/docs/agents/requirement-evaluator.md) |
| 2 | **CaseGeni** | [📐 Architecture](Architecture/docs/architecture/casegeni.md) | [📄 Requirements](Architecture/docs/solutions/casegeni.md) | [🤖 Test Case Generator](Architecture/docs/agents/test-case-generator.md) |
| 3 | **DataGeni** | [📐 Architecture](Architecture/docs/architecture/datageni.md) | [📄 Requirements](Architecture/docs/solutions/datageni.md) | [🤖 Test Data Generator](Architecture/docs/agents/test-data-generator.md) |
| 4 | **Auto-PlayPilot** | [📐 Architecture](Architecture/docs/architecture/auto-playpilot.md) | [📄 Requirements](Architecture/docs/solutions/auto-playpilot.md) | [🤖 Playwright MCP](Architecture/docs/agents/playwright-mcp.md) |
| 5 | **Insights360** | [📐 Architecture](Architecture/docs/architecture/insights360.md) | [📄 Requirements](Architecture/docs/solutions/insights360.md) | [🤖 Report Agent](Architecture/docs/agents/report-agent.md) |
| 6 | **Perf-Xi** | [📐 Architecture](Architecture/docs/architecture/perf-xi.md) | [📄 Requirements](Architecture/docs/solutions/perf-xi.md) | [🤖 Performance Agent](Architecture/docs/agents/performance-agent.md) |
| 7 | **Secure-Xi** | [📐 Architecture](Architecture/docs/architecture/secure-xi.md) | [📄 Requirements](Architecture/docs/solutions/secure-xi.md) | [🤖 Security Agent](Architecture/docs/agents/security-agent.md) |

---

## 📋 Specifications

| Spec | Path | Description |
|---|---|---|
| [Requirements Register](Architecture/specs/requirements/requirements-register.md) | `specs/requirements/` | Consolidated FR/NFR across all 7 solutions |
| [Test Strategy](Architecture/specs/tests/test-strategy.md) | `specs/tests/` | Test pyramid, traceability, coverage targets |
| [Security Specification](Architecture/specs/security/security-spec.md) | `specs/security/` | Threat model, auth, data protection, guardrails |
| [Performance Specification](Architecture/specs/performance/performance-spec.md) | `specs/performance/` | SLIs/SLOs, budgets, capacity planning, runbooks |

---

## 🗂️ Architecture Diagrams (20 SVGs)

All diagrams are in [`assets/images/`](Architecture/assets/images/).

| Diagram | File | Description |
|---|---|---|
| Platform Overview | `01-platform-overview.svg` | QI Offerings end-to-end architecture |
| AI & RAG Pipeline | `02-ai-rag-pipeline.svg` | LLM providers, embeddings, vector store |
| UI/UX Architecture | `03-ui-ux-architecture.svg` | React / Next.js front-end layers |
| Backend Services | `04-backend-services.svg` | FastAPI microservices component view |
| Data & Storage | `05-data-storage.svg` | PostgreSQL, blob, vector store ERD |
| Security Architecture | `06-security-architecture.svg` | Defence-in-depth layers |
| Observability & QE | `07-observability-qe.svg` | OpenTelemetry, metrics, tracing |
| Deployment Topology | `08-deployment-topology.svg` | AKS, CI/CD, blue/green |
| QI Offerings Overview | `qi-offerings-overview.svg` | Solution overview hero image |
| Retrieval Pipeline | `retrieval-pipeline.svg` | RAG retrieval flow detail |
| ZenseAI.Qi Platform | `zenseai-qi-platform.svg` | Solutions, agents, providers, data layer |
| Agent ↔ Solution Mapping | `zenseai-qi-agent-mapping.svg` | 7 agents mapped to solutions & models |
| Roadmap Timeline | `zenseai-qi-roadmap.svg` | 5-phase delivery Mar → Jul |
| DeepSpeci Architecture | `arch-deepspeci.svg` | Requirement ingestion, evaluation, validation |
| CaseGeni Architecture | `arch-casegeni.svg` | Test case generation, dedup, coverage |
| DataGeni Architecture | `arch-datageni.svg` | Synthetic data, PII scan, FK integrity |
| Auto-PlayPilot Architecture | `arch-auto-playpilot.svg` | Playwright codegen, POM, troubleshoot |
| Insights360 Architecture | `arch-insights360.svg` | Aggregation, AI narrative, dashboards |
| Perf-Xi Architecture | `arch-perf-xi.svg` | k6 generation, SLI/SLO, capacity |
| Secure-Xi Architecture | `arch-secure-xi.svg` | SAST/DAST/SCA, threat model, CI gate |

---

## 👤 Persona Quick-Start

| Persona | Recommended Reading Path |
|---|---|
| **Stakeholders / Managers** | [Overview](Architecture/docs/01-overview.md) → [Roles](Architecture/docs/roles.md) → [Insights360](Architecture/docs/architecture/insights360.md) → [Roadmap](Architecture/docs/roadmap/roadmap.md) |
| **Developers** | [Backend](Architecture/docs/05-backend-apis.md) → [AI Integration](Architecture/docs/03-ai-integration.md) → [DeepSpeci Arch](Architecture/docs/architecture/deepspeci.md) → [Appendix](Architecture/docs/11-appendix.md) |
| **QE Engineers** | [Observability](Architecture/docs/08-observability-qe.md) → [Test Strategy](Architecture/specs/tests/test-strategy.md) → [CaseGeni Arch](Architecture/docs/architecture/casegeni.md) |
| **Automation Engineers** | [Auto-PlayPilot Arch](Architecture/docs/architecture/auto-playpilot.md) → [Playwright MCP](Architecture/docs/agents/playwright-mcp.md) → [DataGeni Arch](Architecture/docs/architecture/datageni.md) |
| **Security Engineers** | [Security Spec](Architecture/specs/security/security-spec.md) → [Secure-Xi Arch](Architecture/docs/architecture/secure-xi.md) → [Security Agent](Architecture/docs/agents/security-agent.md) |
| **Performance Engineers** | [Performance Spec](Architecture/specs/performance/performance-spec.md) → [Perf-Xi Arch](Architecture/docs/architecture/perf-xi.md) → [Performance Agent](Architecture/docs/agents/performance-agent.md) |
| **SMEs / Non-Technical** | [Roles](Architecture/docs/roles.md) → [DeepSpeci Req](Architecture/docs/solutions/deepspeci.md) → [Insights360 Req](Architecture/docs/solutions/insights360.md) |

---

## 🏗️ Tech Stack

```
Frontend        React 18 / Next.js 15 · Tailwind CSS · Recharts · Monaco Editor
Backend         Python 3.12 · FastAPI · Node.js 22 (Auto-PlayPilot)
AI Agents       LangChain / LangGraph · Presidio PII
LLM Providers   Gemini · Claude · OpenAI
Data Stores     PostgreSQL 16 · pgvector · TimescaleDB · Redis 7
Object Storage  Azure Blob / S3
Testing         Playwright · k6 · semgrep · OWASP ZAP · Snyk / Trivy
Infrastructure  Docker · Kubernetes (AKS) · GitHub Actions · Bicep IaC
Docs Site       Jekyll · Kramdown · Mermaid.js · GitHub Pages
```

---

## 📁 Repository Structure

```
├── Architecture/
├── _config.yml                         # Jekyll configuration
├── _layouts/
│   └── default.html                    # Page layout with Mermaid.js support
├── index.md                            # 🏠 Architecture Hub (landing page)
├── architecture.md                     # Full E2E architecture (single page)
├── docs/
│   ├── architecture.md                 # Platform architecture + Mermaid diagrams
│   ├── roles.md                        # Roles, personas, RACI matrix
│   ├── 01-overview.md                  # QI Offerings overview
│   ├── 02-architecture-at-a-glance.md  # Context diagram
│   ├── 03-ai-integration.md            # LLM + RAG pipeline
│   ├── 04-ui-ux.md                     # Frontend architecture
│   ├── 05-backend-apis.md              # Backend services & APIs
│   ├── 06-data-storage.md              # Data stores & ERD
│   ├── 07-security-compliance.md       # Security & compliance
│   ├── 08-observability-qe.md          # Observability & QE
│   ├── 09-deployment.md                # Deployment & CI/CD
│   ├── 10-nfr.md                       # Non-functional requirements
│   ├── 11-appendix.md                  # Appendix
│   ├── architecture/                   # 📐 Solution Architecture & Implementation
│   │   ├── deepspeci.md
│   │   ├── casegeni.md
│   │   ├── datageni.md
│   │   ├── auto-playpilot.md
│   │   ├── insights360.md
│   │   ├── perf-xi.md
│   │   └── secure-xi.md
│   ├── solutions/                      # 📄 Solution Requirement Specs
│   │   ├── deepspeci.md
│   │   ├── casegeni.md
│   │   ├── datageni.md
│   │   ├── auto-playpilot.md
│   │   ├── insights360.md
│   │   ├── perf-xi.md
│   │   └── secure-xi.md
│   ├── agents/                         # 🤖 AI Agent Design Specs
│   │   ├── requirement-evaluator.md
│   │   ├── test-case-generator.md
│   │   ├── test-data-generator.md
│   │   ├── playwright-mcp.md
│   │   ├── report-agent.md
│   │   ├── performance-agent.md
│   │   └── security-agent.md
│   └── roadmap/
│       └── roadmap.md                  # 5-phase product roadmap
├── specs/
│   ├── requirements/
│   │   └── requirements-register.md    # Consolidated FR/NFR register
│   ├── tests/
│   │   └── test-strategy.md            # Test strategy & pyramid
│   ├── security/
│   │   └── security-spec.md            # Security specification
│   └── performance/
│       └── performance-spec.md         # Performance SLI/SLO spec
├── assets/
│   ├── css/
│   │   └── style.css                   # Custom styles
│   └── images/                         # 20 SVG architecture diagrams
│       ├── 01-platform-overview.svg
│       ├── 02-ai-rag-pipeline.svg
│       ├── ...
│       ├── arch-deepspeci.svg
│       ├── arch-casegeni.svg
│       ├── ...
│       ├── zenseai-qi-platform.svg
│       ├── zenseai-qi-agent-mapping.svg
│       └── zenseai-qi-roadmap.svg
├── Gemfile                             # Ruby / Jekyll dependencies
├── .gitignore
└── README.md                           # Architecture-level README
└── README.md                               # ← This file (repo root)
```

**Total:** 40+ Markdown documents · 20 SVG diagrams · 7 solution architectures · 7 agent specs · 7 requirement specs · 4 specification documents

---

## 🖥️ Local Development

```bash
# Prerequisites: Ruby ≥ 3.0, Bundler
bundle install

# Serve locally with live reload
bundle exec jekyll serve --livereload

# Open in browser
open http://localhost:4000/Architecture/
```

---

## 🚀 Deploy to GitHub Pages

1. Push to GitHub: `git push arch main`
2. Go to **Settings → Pages → Source** → select `main` branch
3. Site available at: `https://abhilashgiridharan.github.io/ZenseAI.QI.Arch/`

---

## 📊 Document Statistics

| Category | Count |
|---|---|
| Platform-level docs | 14 |
| Solution architecture & implementation docs | 7 |
| Solution requirement specs | 7 |
| AI agent design specs | 7 |
| Specification documents | 4 |
| SVG architecture diagrams | 20 |
| Mermaid inline diagrams | 30+ |
| **Total pages** | **40+** |

---

<p align="center">
  <strong>ZenseAI.Qi</strong> · QI Offerings · Architecture Documentation · 2026
</p>
