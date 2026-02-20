---
layout: default
title: "4. UI/UX"
nav_order: 6
description: "Frontend tech stack, key screens, RAG-specific UX patterns, and latency handling for QI Offerings."
permalink: "/docs/ui-ux/"
---

[← Back to Architecture Hub]({{ '/' | relative_url }})
{: .fs-3 }

# 4. UI/UX

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | **React 19** with **Next.js 15** (App Router, SSR) |
| State | React Server Components + Zustand (client state) |
| Styling | Tailwind CSS 4 + Radix UI primitives |
| Accessibility | WCAG 2.2 AA; `aria-live` for streaming, focus management |
| Testing | Playwright (E2E), Vitest (unit), Storybook (visual) |

---

## Key Screens

1. **Search / Ask** – conversational input, streaming response, source chips.
2. **Sources & Citations** – expandable citation cards with page-level deep links.
3. **Test & Reports** – quality dashboards, retrieval metrics, test run history.
4. **Admin** – index management, prompt template editor, user roles.

---

## UX Patterns for RAG

| Pattern | Implementation |
|---|---|
| **Source chips** | Inline pill badges `[Source 1]` linking to the original document |
| **Expandable citations** | Accordion below answer showing chunk text + highlight |
| **Confidence gauge** | Circular gauge (green ≥ 0.8, amber 0.6–0.8, red < 0.6) |
| **Streaming tokens** | Server-Sent Events rendered token-by-token |
| **Skeleton loaders** | Animated placeholders during retrieval (~300 ms) |
| **Empty / error states** | Friendly illustrations + suggested queries for empty; retry + contact for errors |

---

## Latency Handling

- **Optimistic UI** – show user query immediately in chat history.
- **SSE streaming** – first token appears in < 800 ms; full answer streams over 1–3 s.
- **Abort controller** – user can cancel in-flight requests.

---

**Previous:** [← AI Integration]({{ '/docs/ai-integration/' | relative_url }}) · **Next:** [Backend & APIs →]({{ '/docs/backend-apis/' | relative_url }})
