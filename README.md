# MyWiki — Personal LLM Wiki v3

A small, durable, Markdown-based engineering wiki that acts as the stable control layer for Hermes and coding agents.

## What this wiki is

- Global FE/BE engineering guidelines.
- Project context and decisions.
- Reusable golden code examples (links, not dumps).
- Distilled research conclusions.
- Task packets that define safe Hermes work boundaries.

## What this wiki is not

- Not a second brain.
- Not an autonomous memory system (Hermes owns memory).
- Not a central task manager (GitHub/Linear owns large backlogs).
- Not a custom RAG system.
- Not a Hermes skill replacement (Hermes Curator owns skill lifecycle).

## How to use it with Hermes

1. Hermes reads `index.md` first, then the active project's `context.md` and `linked-guidelines.md`.
2. Non-trivial work runs through a task packet under `projects/<project>/task-packets/`.
3. Hermes default mode is `plan-only`. Implementation modes need explicit human approval.
4. Hermes wiki proposals go only to `inbox/hermes-proposals/`. Canonical edits require human approval.
5. Branch naming: `hermes/<project>-<task>`. No push to `main`.

## How to add a project

1. Create `projects/<project>/` with `context.md`, `linked-guidelines.md`, `decisions.md`, `current-state.md`, `repo-map.md`, `backlog.md`, and `task-packets/`.
2. Link the global guidelines that apply.
3. Seed at least 5 accepted decisions from real project facts.
4. Create one task packet (default `plan-only`).
5. Run one Hermes dry run before allowing implementation modes.

Do not bulk-migrate. Add projects only after the previous project's loop has succeeded.

## How to promote research

1. Capture in `research/inbox/` or `inbox/quick-capture/`.
2. Distill into `research/accepted/` only when the note has Question, Keep, Decision, Rejected, Applies To, Next Action, and Sources.
3. If a research note has no decision and no next action, it stays non-canonical.

## Maintenance

See `meta/maintenance-checklist.md`. Weekly 15 minutes, monthly 30 minutes, quarterly 1-2 hours. If quarterly prune is skipped twice, pause expansion until the wiki is cleaned.

## Out of launch scope

`ops/`, `vendors/`, `runbooks/`, `postmortems/`, `prompts/`, `glossary.md`, global `TASKS.md`, qmd, custom vector DB, mobile capture app, wiki publishing site. Add only when triggers documented in `llm-wiki-plan-v3.md` appear.
