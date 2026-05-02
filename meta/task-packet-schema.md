---
title: Task Packet Schema
summary: Hermes work-order contract. Defines required sections, modes, and stop conditions.
type: meta
status: canonical
updated: 2026-05-02
---

# Task Packet Schema

A task packet is a Hermes work order. It tells Hermes exactly what to do, where to do it, and when to stop.

## Frontmatter

```yaml
---
id: <project>-YYYYMMDD-short-name
project: <project>
repo_path: /Users/eloelo/Downloads/<Repo>
status: draft | approved | running | review | done | cancelled
mode: research-only | plan-only | implement-local | implement-pr | review-only | wiki-proposal
risk: low | medium | high
created: YYYY-MM-DD
owner: human
executor: hermes
---
```

Default `mode` is `plan-only`. Implementation modes (`implement-local`, `implement-pr`) require explicit human approval per packet.

## Required Sections

Every task packet MUST include:

- **Goal** — one paragraph.
- **Why** — product or technical reason.
- **End Goal** — the precise outcome Hermes optimizes for.
- **Output Format** — exact deliverable (report, branch, PR, task packet, diff summary, wiki proposal).
- **Recommended Approach** — preferred sequence of work. Hermes may propose a safer alternative before execution.
- **Common Failure Handling** — known traps and the required behavior when they appear.
- **Allowed Paths** — exact filesystem prefixes Hermes may edit. Hermes MUST stop if the task wants edits outside this list.
- **Required Context** — wiki and repo files Hermes must read before acting.
- **Non-Goals** — what Hermes MUST NOT do.
- **Stop Conditions** — explicit reasons Hermes must halt and ask the human.
- **Output Required** — list of items Hermes must return on completion.

## Conditionally Required Sections

These are required only when the mode involves writing code:

- **Reference Files In Repo** — required for `implement-local`, `implement-pr`, `review-only`.
- **Implementation Plan** — required for `implement-local`, `implement-pr`.
- **Verification** — required for `implement-local`, `implement-pr` (build/test commands with pass criteria).

For `research-only`, `plan-only`, and `wiki-proposal`, these sections are optional or replaced by a research-output description.

## Default Stop Conditions

Stop and ask the human if any are true:
- Task wants paths outside `Allowed Paths`.
- Dependency changes are needed.
- Secrets are needed and not declared.
- Build requires unrelated fixes.
- Architecture differs from the linked guidelines.
- More than 3 unexpected files need edits.

## Output Required (default)

- Branch name (or `none` for non-implementation modes).
- Files changed (or `none`).
- Summary.
- Commands run and results.
- Architecture compliance against linked guidelines.
- Risks / open questions.
- Suggested wiki updates (these go only to `inbox/hermes-proposals/`).

## Hermes Report Format

```md
## Summary
## Files Changed
## Verification
## Architecture Compliance
## Risks / Open Questions
## Suggested Wiki Updates
```
