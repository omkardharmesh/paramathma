---
title: Saarthi Project Context
summary: Backend and data-control-plane for Saarthi’s spiritual chatbot, retrieval, Supabase auth, corpus, deployment APIs, and workflows.
type: project-context
status: draft
updated: 2026-05-14
---

# Saarthi — Project Context

## Product

- One-liner: Backend and data-control-plane for Saarthi’s spiritual chatbot, retrieval, Supabase auth, corpus, deployment APIs, and workflows.
- Product role: Backend-only repo for Saarthi Chat and product backend/data-control-plane work.
- User-facing app: Saarthi spiritual journey product; Saarthi Chat is one tab inside the larger app.
- Audience: Hindi, English, and Hinglish spiritual guidance users; backend clients include the future `saarthi-fe` frontend.
- Monetization model: TBD — ask user to confirm; payment processing is planned through Razorpay.
- Status: In development; backend POC exists, public product-auth readiness is pending.

## Repo

- Path: `/Users/eloelo/Downloads/chatbot/saarthi`
- Remote: `https://github.com/omkardharmesh/saarthi-be.git`
- Package root: `/Users/eloelo/Downloads/chatbot/saarthi/src/saarthi`
- Python package: `saarthi`
- Frontend counterpart: `/Users/eloelo/Downloads/chatbot/saarthi-fe` — onboard separately; do not treat Chainlit as production frontend.
- Main docs: `/Users/eloelo/Downloads/chatbot/saarthi/README.md`, `/Users/eloelo/Downloads/chatbot/saarthi/docs/tech-stack.md`, `/Users/eloelo/Downloads/chatbot/saarthi/docs/backend-repo-structure.md`

## Stack

- Language/runtime: Python 3.12.
- Package manager/build: `uv` with `uv.lock` and `uv_build`.
- API framework: FastAPI with SSE `/chat` endpoint and `/health` readiness probe.
- Dev/internal UI: Chainlit mounted at `/chat-ui`; production frontend lives in `saarthi-fe`.
- Database: PostgreSQL 17 + pgvector 0.8.2; local Docker DB and Supabase Postgres product target.
- Auth: Supabase Auth with Google Sign-In for MVP; FastAPI JWT verification is required before protected backend calls.
- Migrations: Alembic for current local POC history; `supabase/migrations/` for Supabase demo/product DB direction.
- Retrieval: Hybrid dense/vector + FTS retrieval with pgvector and RRF.
- Response LLM: LiteLLM route `saarthi-default` to DeepInfra `google/gemma-4-31B-it`.
- Embeddings: OpenRouter `google/gemini-embedding-2-preview` with `dimensions: 1024`.
- Reranker: Hosted Jina `jina-reranker-v3` for deploy/runtime; local BGE FP32 only for parity/fallback.
- Tools: Docker, pytest, ruff, SQLAlchemy asyncio, Pydantic Settings, RAGAS eval harness.

## Build Commands

- Install deps: `rtk uv sync --frozen`
- Start local DB: `rtk docker compose up -d db`
- Apply migrations: `rtk uv run alembic upgrade head`
- Run API locally: `rtk uv run uvicorn saarthi.main:app --host 127.0.0.1 --port 8000`
- Run app container: `rtk docker compose up --build saarthi`
- Health check: `rtk curl -sf http://127.0.0.1:8000/health`
- Test suite: `rtk uv run pytest tests/ -q`
- Migration round-trip test: requires `POSTGRES_DSN_TEST`; exact command TBD — ask user to confirm preferred safe DSN handling.
- Deploy: Dockerfile exists for Railway/AWS App Runner/Fly targets; exact deploy commands TBD — ask user to confirm.

## Hard Constraints

- Backend boundary: This repo is backend/data-control-plane only; do not onboard or edit `saarthi-fe` as part of Saarthi backend tasks.
- Frontend boundary: Chainlit is internal/dev UI only, not the production product frontend.
- Product data: Supabase is the source of truth for Auth, product Postgres, pgvector corpus, and RLS.
- Railway rule: Railway Postgres is disposable QA only; do not make it product data storage.
- Auth rule: Do not create a separate auth-service repo unless Supabase Auth becomes a real blocker.
- MVP auth: Google Sign-In only; phone auth is P2, but backend schema planning keeps phone fields nullable.
- User identity: `auth.users.id` is canonical; do not use phone number as primary identifier.
- Retrieval quality: Do not change embedding/reranker strategy or disable reranking without golden-query eval approval.
- Rashi/Kundli: P2 external-provider display adapters only; do not implement in-house astrology logic in MVP.
- Secrets: Never store raw API keys, database passwords, JWTs, service-role keys, or `.env` contents in MyWiki, git, or chat.

## What Not To Touch

- Off-limits repo: `/Users/eloelo/Downloads/chatbot/saarthi-fe` until frontend onboarding is requested.
- Off-limits secrets: `/Users/eloelo/Downloads/chatbot/saarthi/.env` and any raw credential files.
- Off-limits local metadata: `/Users/eloelo/Downloads/chatbot/saarthi/.idea/` unless the user explicitly asks.
- Off-limits product data changes: Product-specific Supabase tables, RLS policies, and Edge Functions without approved backend requirements.
- Off-limits migration drift: Do not maintain Alembic and `supabase/migrations/` as two independent production authorities.
- Off-limits UI assumption: Do not treat Chainlit branding or `/chat-ui` as production frontend work.

## External Services

- Supabase: Auth, Postgres, pgvector, corpus storage, RLS; target region `ap-south-1` Mumbai.
- DeepInfra: Response LLM via `DEEPINFRA_API_KEY` and LiteLLM route `saarthi-default`.
- OpenRouter: Gemini Embedding 2 Preview via `OPENROUTER_API_KEY`.
- Jina AI: Hosted reranker via `JINA_API_KEY` and `JINA_RERANKER_MODEL`.
- Sarvam: Optional LLM/TTS routes via `SARVAM_API_KEY`; current production role TBD — ask user to confirm.
- Razorpay: Planned payments SDK + Supabase Edge Function webhook.
- Upstash Redis: Planned rate limit, semantic cache, and daily turn caps.
- Langfuse: Planned observability with PII redaction before logging.
- Railway: Demo FastAPI/Chainlit hosting only; nearest region Singapore.
- AWS App Runner: Public beta FastAPI target in Mumbai while AWS credits are active.
- Fly.io: Mumbai exit/backup hosting path.
- Rashi/Kundli provider: TBD — ask user to confirm P2 external provider.

## Current Release Status

- See `current-state.md`.
