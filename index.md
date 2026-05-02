---
title: LLM Wiki Index
summary: First-read map for all agents. Links to active projects, global guidelines, and ignore rules.
type: meta
status: canonical
updated: 2026-05-02
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

- [Yawnly](projects/yawnly/context.md) — Compose Multiplatform bedtime story app. First Hermes pilot.

## Global Guidelines

### Frontend
- [Compose Multiplatform architecture](guidelines/frontend/compose-multiplatform.md)
- [Compose UI conventions](guidelines/frontend/compose-ui.md)

### Backend
- [Supabase usage](guidelines/backend/supabase.md)
- [Supabase Edge Functions](guidelines/backend/supabase-edge-functions.md)

### Agents
- [Hermes delegation](guidelines/agents/hermes-delegation.md)

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
