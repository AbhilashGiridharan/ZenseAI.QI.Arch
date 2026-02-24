---
layout: default
title: "9. Deployment & Environments"
nav_order: 11
description: "Containers, AKS, CI/CD with GitHub Actions, Bicep IaC, and deployment topology."
permalink: "/docs/deployment/"
---

[← Back to Architecture Hub](../index.md)
{: .fs-3 }

# 9. Deployment & Environments

---

## Architecture Diagram

![Deployment & CI/CD — Architecture](../assets/images/08-deployment-topology.svg)

---

## Environment Topology

| Environment | Purpose | Infra |
|---|---|---|
| `dev` | Feature development, rapid iteration | Single-node AKS, shared DB |
| `staging` | Integration, QE, performance testing | Multi-node AKS, isolated DB |
| `production` | Live traffic | Multi-node AKS (3 zones), HA DB |

---

## Packaging

- All services containerised with **multi-stage Dockerfiles** (slim Python 3.12 base).
- Images pushed to **Azure Container Registry** (geo-replicated).

---

## Orchestration

- **Azure Kubernetes Service (AKS)** with managed node pools.
- Horizontal Pod Autoscaler (HPA) on CPU + custom metric (request queue depth).
- Ingress via **NGINX Ingress Controller** (TLS termination).

---

## CI/CD – GitHub Actions

```yaml
# .github/workflows/deploy.yml (simplified)
name: Build & Deploy
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: docker/build-push-action@v6
        with:
          push: true
          tags: ${{ env.ACR_LOGIN_SERVER }}/qi-api:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: azure/k8s-deploy@v5
        with:
          manifests: k8s/
          images: ${{ env.ACR_LOGIN_SERVER }}/qi-api:${{ github.sha }}
          strategy: blue-green
```

---

## Infrastructure as Code

- **Bicep** templates in `/infra` for all Azure resources.
- Modules: `network.bicep`, `aks.bicep`, `ai-search.bicep`, `openai.bicep`, `postgres.bicep`, `monitoring.bicep`.
- `bicep param` files per environment (`dev.bicepparam`, `staging.bicepparam`, `prod.bicepparam`).

---

## Deployment Diagram

```mermaid
flowchart TD
    CLIENT([Browser / Mobile]) -->|HTTPS| CDN[Azure Front Door<br/>+ CDN]
    CDN --> WAF[WAF v2]
    WAF --> APIM[Azure API<br/>Management]
    APIM --> ING[NGINX Ingress]

    subgraph AKS["AKS Cluster"]
        ING --> AUTH_POD[Auth Pod]
        ING --> RET_POD[Retrieval Pod]
        ING --> ORCH_POD[Orchestration Pod]
        ING --> RPT_POD[Reporting Pod]
        ING --> ING_POD[Ingestion Worker]
    end

    AUTH_POD & RET_POD & ORCH_POD & RPT_POD --> PG[(PostgreSQL<br/>Flexible Server)]
    RET_POD & ORCH_POD --> VEC[(Azure AI Search)]
    ORCH_POD --> LLM[Azure OpenAI]
    ING_POD --> BLOB[(Blob Storage)]
    ING_POD --> SB[[Service Bus]]

    subgraph Observability
        OTEL[OTel Collector]
        APPINS[Application Insights]
        GRAF[Grafana]
    end

    AKS -.->|traces / logs| OTEL --> APPINS --> GRAF
```

---

**Previous:** [← Observability & QE](08-observability-qe.md) · **Next:** [Non-Functional Requirements →](10-nfr.md)
