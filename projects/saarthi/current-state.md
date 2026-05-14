---
title: Saarthi Current State
summary: Backend POC exists; MyWiki project context is drafted; product auth and payment gates remain before public beta.
type: current-state
status: draft
updated: 2026-05-14
---

# Saarthi — Current State

## Working

- Repo exists at `/Users/eloelo/Downloads/chatbot/saarthi`.
- Remote exists at `https://github.com/omkardharmesh/saarthi-be.git`.
- FastAPI app exists with `/health`, `/chat`, and mounted Chainlit `/chat-ui` internal UI.
- Local Docker Compose includes PostgreSQL 17 + pgvector 0.8.2.
- Alembic migrations and Supabase migration workspace exist.
- README documents local quickstart, tests, endpoints, and repo layout.
- `docs/tech-stack.md` is the current product stack source of truth.
- Dockerfile exists for containerized deploy targets.

## In Progress

- MyWiki project files are drafted under `projects/saarthi/` and awaiting human review or commit.
- Backend/data-control-plane scope is documented separately from the future `saarthi-fe` frontend project.
- Product flow treats Saarthi Chat as one tab inside a larger 30-day spiritual journey app.
- Supabase Auth and product backend schema work need approved requirements before public product traffic.

## Pending / Blocked

- Monetization model is TBD — ask user to confirm.
- Exact production deploy commands are TBD — ask user to confirm after target is chosen.
- `/chat` still needs Supabase JWT verification and `user_id = jwt.sub` for protected backend calls.
- Subscription gate before retrieval/LLM calls is pending.
- Redis-backed daily quota/rate limiting is pending.
- Product-specific Supabase tables/RLS policies require approved backend requirements.
- Migration authority needs final decision before production: `supabase/migrations/` versus Alembic.
- Frontend counterpart `/Users/eloelo/Downloads/chatbot/saarthi-fe` is not onboarded yet.

## Risks

- Two migration histories can drift if Alembic and `supabase/migrations/` both evolve independently.
- Chainlit can be mistaken for production frontend if the backend/frontend boundary is ignored.
- Retrieval quality can regress if embedding/reranker choices change without golden-query eval approval.
- Product data can leak or fragment if Railway Postgres is used beyond disposable QA.
- Secrets can leak if `.env` values or service-role keys are copied into docs or prompts.

## Next Steps

1. Confirm monetization model and any additional do-not-touch areas.
2. Keep Saarthi backend and `saarthi-fe` frontend onboarding separate.
3. Create approved backend requirements before product auth/account/payment migrations.
4. Decide production migration authority before public beta.
5. Smoke test `/chat` SSE on the selected deploy target.
