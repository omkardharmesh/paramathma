---
title: AGENTS.md (MyWiki root)
summary: Shared base instructions for any coding agent reading this wiki.
type: meta
status: canonical
updated: 2026-05-02
---

# AGENTS.md — Shared Base Instructions

This file applies to every coding agent (Hermes, Claude, Cursor, Codex, etc.) that reads this wiki.

## Read Protocol

1. Read `index.md` first.
2. Read the active project's `context.md` and `linked-guidelines.md`.
3. Read only the linked guideline and golden-example pages required for the current task.
4. Read the current task packet.

Do not recursively load the wiki. Do not bulk-read `research/`, `inbox/`, `golden-examples/`, archived/stale pages, or task packets that are not linked from the current project context. See `meta/wikiignore.md`.

## Work Lanes

- Non-trivial work requires an approved task packet. See `meta/task-packet-schema.md`.
- Default task packet mode is `plan-only`. Implementation modes need explicit human approval.
- Wiki proposals go to `inbox/hermes-proposals/` (Hermes) or `inbox/claude-proposals/` (Claude/Cursor).
- Do not write canonical guidelines or project decisions without human approval.
- Do not modify page frontmatter only to record that a page was read.

## Stop Conditions

Stop and ask the human if any of these are true:
- Task needs paths outside the task packet's `Allowed Paths`.
- Dependency changes are required.
- Secrets are needed.
- Build requires unrelated fixes.
- Architecture differs from the linked guidelines.
- More than 3 unexpected files need edits.

## Secrets

Never store raw secrets in the wiki. Allowed: env var names, Supabase secret names, `op://` references, Bitwarden item names/IDs. Forbidden: API keys, tokens, JWTs, service role keys, private keys, `.env` contents.

## Reports

When a task packet finishes, return:
- Summary
- Files Changed
- Verification (commands run + results)
- Architecture Compliance (against linked guidelines)
- Risks / Open Questions
- Suggested Wiki Updates (if any) — these go to the proposals inbox, never directly into canonical pages.
