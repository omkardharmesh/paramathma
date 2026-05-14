---
title: LLM Wiki Index
summary: First-read map for all agents. Links to active projects, global guidelines, and ignore rules.
type: meta
status: canonical
updated: 2026-05-14
---

# LLM Wiki Index

This wiki is the stable control layer for Hermes and coding agents. Hermes owns execution, memory, skills, sessions, cron, and Telegram. The wiki only owns stable truths.

## First Read Protocol

1. Read this file (`index.md`).
2. Read the active project's `context.md`.
3. Read the active project's `linked-guidelines.md`.
4. Read only the linked guideline and golden-example pages required by the task.
5. Read the current task packet under `projects/<project>/task-packets/`.

Agents must not recursively load the wiki. See `meta/wikiignore.md`.

## Active Projects

- [Kokoros](projects/kokoros/context.md) — Self-hosted Kokoro TTS service for Yawnly via Supabase and Cloudflare.
- [Saarthi](projects/saarthi/context.md) — Backend and data-control-plane for Saarthi’s spiritual chatbot, retrieval, Supabase auth, corpus, deployment APIs, and workflows.
- [Yawnly](projects/yawnly/context.md) — Compose Multiplatform bedtime story app. First Hermes pilot.
- [Yawnly Marketing](projects/yawnly-marketing/context.md) — Post-launch marketing research, launch plans, creative assets, and store listing strategy for Yawnly.

## Global Guidelines

### Frontend
- [Compose Multiplatform architecture](guidelines/frontend/compose-multiplatform.md)
- [CMP feature scaffold](guidelines/frontend/cmp-feature-scaffold.md)
- [Compose UI conventions](guidelines/frontend/compose-ui.md)

### Backend
- [Supabase usage](guidelines/backend/supabase.md)
- [Supabase Edge Functions](guidelines/backend/supabase-edge-functions.md)

### Agents
- [Hermes delegation](guidelines/agents/hermes-delegation.md)
- [Spec workflow](guidelines/agents/spec-workflow.md)
- [CMP PR review](guidelines/agents/cmp-pr-review.md)
- [MyWiki project onboarding](guidelines/agents/mywiki-project-onboarding.md)

## Schemas

- [Page schema](meta/page-schema.md)
- [Task packet schema](meta/task-packet-schema.md)
- [Research note schema](meta/research-note-schema.md)
- [Maintenance checklist](meta/maintenance-checklist.md)

## Do Not Bulk Load

See `meta/wikiignore.md`.

## Write Lanes

- Hermes proposals → `inbox/hermes-proposals/`
- Claude/Cursor proposals → `inbox/claude-proposals/`
- Quick capture → `inbox/quick-capture/`
- Canonical guidelines/decisions require human approval.
