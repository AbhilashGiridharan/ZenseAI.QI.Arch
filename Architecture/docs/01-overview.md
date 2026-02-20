---
layout: default
title: "1. Overview"
nav_order: 3
description: "Problem statement, success criteria, non-goals, constraints, and capability map for QI Offerings."
permalink: "/docs/overview/"
---

[← Back to Architecture Hub]({{ '/' | relative_url }})
{: .fs-3 }

# 1. Overview

---

## Problem Statement

Teams spend excessive time manually searching, interpreting, and cross-referencing large document corpora to answer domain-specific questions. Existing keyword search yields low-precision results and provides no contextual synthesis.

---

## Success Criteria

- **Answer relevance** ≥ 90 % (human-evaluated on a golden dataset).
- **Retrieval precision@5** ≥ 0.85.
- **p95 response latency** ≤ 3 s for single-turn queries.
- **Zero PII leakage** in LLM responses (validated by content filters).
- Full audit trail for every query ↔ response pair.

---

## Non-Goals

- Replacing human expert review for regulatory submissions.
- Real-time streaming of external third-party data feeds.
- Training or fine-tuning foundation models in-house (use managed APIs).

---

## Constraints

- All data must reside in **Azure regions** compliant with corporate data-residency policy.
- LLM calls must route through **Azure OpenAI** (no direct OpenAI endpoint).
- Budget ceiling for Azure AI Search: **S1 tier** (≤ 50 GB index).

---

## High-Level Capability Map

- 📄 **Document Ingestion** – parse, chunk, embed, index.
- 🔍 **Hybrid Retrieval** – vector + lexical search with metadata filters.
- 🤖 **LLM Orchestration** – prompt assembly, safety filters, streaming.
- 🖥️ **Interactive UI** – conversational search with citations and confidence.
- 🔐 **Security & Compliance** – Entra ID, encryption, audit.
- 📊 **Observability** – distributed tracing, token metrics, quality dashboards.
- 🚀 **CI/CD & IaC** – GitHub Actions, Bicep, blue/green deploys.

---

**Next:** [Architecture at a Glance →]({{ '/docs/architecture-at-a-glance/' | relative_url }})
