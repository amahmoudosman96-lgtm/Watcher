# AGENTS.md — Watcher build loop

This repo is built with an AI-agent-assisted loop (Claude Code / Codex implement; the founder writes specs,
reviews, integrates, and tests against real WhatsApp traffic). Read this before implementing.

## Sources of truth (read first, every time)

1. `docs/build-spec-addendum.md` — engineering decisions, data model, and the resolution of the v1.2
   identity-resolution contradiction. **Do not re-derive architecture; it's settled here.**
2. `DESIGN-SPEC.md` — every control-page screen is built against this. Never invent styles per screen; never
   hardcode a color — reference a token. Propose new tokens in the spec first.
3. The MVP v1.2 PDF — product/strategy intent (the "why").

## Rules

- **One language server-side: Python.** TypeScript only in `apps/control-page` (framework constraint, v1.2 §11).
- **Pydantic v2 schemas are the single source of truth** — the same models back the LLM structured output, the
  DB row, and the REST contract. Build them first.
- **Eval-tool-first discipline:** no prompt change merges without an eval run. CI blocks merges that drop overall
  accuracy >2pp (v1.2 §12).
- **Multi-tenancy is non-negotiable:** every business table carries `tenant_id` with RLS. Never write a query
  that can cross tenants.
- **Self-hosted constraint:** no runtime external CDN/font/icon calls anywhere — the regulated tier forbids data
  egress (v1.2 §10).
- **Nothing from the marketing website** is copied into this repo.

## Build order

See `docs/build-spec-addendum.md` §18. Briefly: schemas → webhook receiver → classifier + eval baseline →
control page → media pipeline → identity resolution → rules → dedup sweep.

## Spec-per-deliverable

Each deliverable starts from a 1–3 page spec naming the module boundary, the input/output schema, the error
cases, and the test fixtures. Vague specs cost three iterations; precise specs land in one (v1.2 §14).
