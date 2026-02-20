---
layout: default
title: "QI Offerings – Architecture Hub"
nav_order: 1
description: "Architecture documentation hub for the QI Offerings platform."
permalink: "/"
---

# QI Offerings – Architecture Hub

Welcome to the **QI Offerings** architecture documentation site. This hub provides comprehensive technical references for the end-to-end platform.

---

## 📚 Documentation

| Document | Description |
|---|---|
| [E2E Architecture (LLM + RAG)]({{ '/architecture/' | relative_url }}) | Full architecture covering AI integration, UI/UX, backend, security, observability, and deployment |

---

## Quick Links

- **AI Integration** – [LLM + RAG pipeline details]({{ '/architecture/#3-ai-integration-llm--rag' | relative_url }})
- **UI/UX** – [React / Next.js frontend]({{ '/architecture/#4-uiux' | relative_url }})
- **Backend** – [FastAPI services & APIs]({{ '/architecture/#5-backend--apis' | relative_url }})
- **Data** – [Storage & ERD]({{ '/architecture/#6-data--storage' | relative_url }})
- **Security** – [Identity, network, data protection]({{ '/architecture/#7-security--compliance' | relative_url }})
- **Observability** – [Metrics, tracing, QE]({{ '/architecture/#8-observability--quality-engineering-qe' | relative_url }})
- **Deployment** – [CI/CD, IaC, environments]({{ '/architecture/#9-deployment--environments' | relative_url }})

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
