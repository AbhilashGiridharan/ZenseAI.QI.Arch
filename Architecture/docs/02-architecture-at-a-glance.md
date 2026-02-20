---
layout: default
title: "2. Architecture at a Glance"
nav_order: 4
description: "High-level context diagram showing actors, UI, gateway, backend, AI, data, and observability layers."
permalink: "/docs/architecture-at-a-glance/"
---

[← Back to Architecture Hub](../index.md)
{: .fs-3 }

# 2. Architecture at a Glance

> Context diagram showing how actors interact with the platform through the UI, API gateway, backend services, AI layer, data stores, and observability stack.

---

## Context Diagram

```mermaid
flowchart TB
    subgraph Actors
        A1([Stakeholder])
        A2([Developer])
        A3([QE Engineer])
    end

    subgraph UI["UI – React / Next.js"]
        U1[Search & Ask]
        U2[Reports & Admin]
    end

    subgraph Gateway["Azure API Management"]
        GW[API Gateway<br/>Auth · Rate-limit · Cache]
    end

    subgraph Backend["Backend Services – FastAPI"]
        S1[Auth Service]
        S2[Retrieval Service]
        S3[Orchestration Service]
        S4[Reporting Service]
    end

    subgraph AI["AI Integration"]
        VEC[(Azure AI Search<br/>Vector Store)]
        LLM[Azure OpenAI<br/>gpt-4.1 / 4o]
    end

    subgraph Data["Data Stores"]
        PG[(PostgreSQL)]
        BLOB[(Azure Blob<br/>Storage)]
    end

    subgraph Obs["Observability"]
        OT[OpenTelemetry<br/>Collector]
        MON[Azure Monitor /<br/>Grafana]
    end

    A1 & A2 & A3 --> U1 & U2
    U1 & U2 -->|HTTPS| GW
    GW --> S1 & S2 & S3 & S4
    S2 -->|query| VEC
    S3 -->|prompt + context| LLM
    S2 & S3 --> PG
    S2 --> BLOB
    S1 & S2 & S3 & S4 -->|traces, logs| OT
    OT --> MON
```

---

## Layer Summary

| Layer | Technology | Role |
|---|---|---|
| **UI** | React 19 / Next.js 15 | Conversational search, reports, admin |
| **Gateway** | Azure API Management | Auth, rate-limiting, caching |
| **Backend** | Python FastAPI | Auth, retrieval, orchestration, reporting |
| **AI** | Azure OpenAI + Azure AI Search | LLM completion, vector/hybrid search |
| **Data** | PostgreSQL + Blob Storage | Relational data, raw document storage |
| **Observability** | OpenTelemetry + Azure Monitor | Distributed tracing, metrics, logs |

---

**Previous:** [← Overview](01-overview.md) · **Next:** [AI Integration →](03-ai-integration.md)
