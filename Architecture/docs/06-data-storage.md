---
layout: default
title: "6. Data & Storage"
nav_order: 8
description: "Vector DB, primary database, object store, and entity-relationship diagram for QI Offerings."
permalink: "/docs/data-storage/"
---

[← Back to Architecture Hub](../index.md)
{: .fs-3 }

# 6. Data & Storage

---

## Architecture Diagram

![Data & Storage — Architecture](../assets/images/05-data-storage.svg)

---

## Storage Map

| Store | Technology | Purpose |
|---|---|---|
| Vector index | Azure AI Search (S1) | Chunk embeddings + metadata, hybrid retrieval |
| Primary DB | PostgreSQL 16 (Azure Flexible Server) | Users, sessions, queries, responses, audit |
| Object store | Azure Blob Storage (Hot tier) | Raw documents (PDF, DOCX, HTML) |

---

## Entity-Relationship Diagram

```mermaid
erDiagram
    DOCUMENT ||--o{ CHUNK : "split into"
    CHUNK ||--o{ CITATION : "referenced by"
    SESSION ||--o{ QUERY : "contains"
    QUERY ||--|| RESPONSE : "produces"
    RESPONSE ||--o{ CITATION : "includes"
    USER ||--o{ SESSION : "owns"

    DOCUMENT {
        uuid id PK
        string title
        string source_url
        string doc_type
        string blob_path
        timestamp ingested_at
    }

    CHUNK {
        uuid id PK
        uuid document_id FK
        int chunk_index
        text content
        int page_number
        jsonb metadata
    }

    SESSION {
        uuid id PK
        uuid user_id FK
        timestamp created_at
    }

    QUERY {
        uuid id PK
        uuid session_id FK
        text query_text
        jsonb filters
        timestamp asked_at
    }

    RESPONSE {
        uuid id PK
        uuid query_id FK
        text answer_text
        float confidence
        int prompt_tokens
        int completion_tokens
        timestamp answered_at
    }

    CITATION {
        uuid id PK
        uuid response_id FK
        uuid chunk_id FK
        float relevance_score
    }

    USER {
        uuid id PK
        string display_name
        string email
        string role
    }
```

---

## Key Design Decisions

- **PostgreSQL** chosen for ACID compliance, JSON support (`jsonb`), and mature Azure managed service.
- **Azure AI Search** provides built-in hybrid (vector + BM25) retrieval without managing a separate vector DB.
- **Blob Storage** (Hot tier) for fast retrieval of source documents during citation lookup.
- All stores connected via **Private Endpoints** (no public access).

---

**Previous:** [← Backend & APIs](05-backend-apis.md) · **Next:** [Security & Compliance →](07-security-compliance.md)
