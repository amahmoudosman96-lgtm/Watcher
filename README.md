# Watcher

The WhatsApp intelligence layer that turns conversations into structured CRM records.

Watcher watches a connected business WhatsApp number, classifies each incoming message with a frontier LLM
(intent, person, company, confidence), and routes it as a structured record to the destination the client
already uses — a Google Sheet or any HTTPS webhook (HubSpot, Pipedrive, Notion, Airtable via recipes).
LLM-first, Arabic from day one, human-in-the-loop, with a real data-residency story for regulated GCC clients.

> This is the **product** repo. The Automera marketing website is a separate repo; nothing is shared.

## Documents

- **`docs/build-spec-addendum.md`** — build-ready engineering decisions resolving the gaps in MVP v1.2
  (auth, multi-tenancy, data model, media pipeline, identity resolution, control-chat state machine, eval,
  residency). Start here before writing backend code.
- **`DESIGN-SPEC.md`** — source of truth for the control-page UI (tokens, type, components, the four views).
  Read before writing any frontend code.
- **`AGENTS.md`** — how the AI-agent build loop works in this repo.

## Stack (locked for v1)

Python 3.12+ end-to-end server-side (FastAPI, Pydantic v2, SQLAlchemy 2.0 + Alembic, Postgres). Meta
WhatsApp Business Cloud API ingestion via `pywa`. Anthropic Claude primary / OpenAI secondary; Qwen via
vLLM for the self-hosted tier. Next.js 15 + TypeScript + Tailwind for the control page.

## Status

Pre-Phase-1. The two spec documents above are complete; the build starts once the §17 "blocks starting the
build" questions in the addendum are answered.
