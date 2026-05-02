---
title: Yawnly Task Packets
summary: How task packets work in Yawnly. Status meanings, allowed modes, review rules.
type: meta
status: canonical
updated: 2026-05-02
---

# Yawnly — Task Packets

Task packets are Hermes work orders. Schema lives in `meta/task-packet-schema.md`.

## Status Meanings

| status      | meaning                                                              |
| ----------- | -------------------------------------------------------------------- |
| `draft`     | Authored but not yet approved by the human.                          |
| `approved`  | Human has approved the packet for execution in the declared mode.    |
| `running`   | Hermes is actively working on it.                                    |
| `review`    | Hermes returned a report; awaiting human review.                     |
| `done`      | Reviewed and accepted.                                               |
| `cancelled` | Closed without execution.                                            |

## Allowed Modes (per packet)

- `research-only` — search/read/summarize. No code edits. No wiki canonical edits.
- `plan-only` — produce a spec or follow-up task packet. No code edits.
- `implement-local` — edit allowed paths on a branch (`hermes/yawnly-<task>`). No push.
- `implement-pr` — edit, commit, push branch, open PR.
- `review-only` — review a branch/diff and report findings.
- `wiki-proposal` — propose wiki updates only into `inbox/hermes-proposals/`.

Default mode for a new packet is `plan-only`. Implementation modes require explicit human approval.

## Allowed Paths Rule

Every packet declares `Allowed Paths`. Hermes must stop and ask the human before editing outside those paths. Hermes must also stop if more than 3 unexpected files need edits, even inside Allowed Paths.

## Stop Conditions (default)

Stop and ask the human if:
- Task wants paths outside `Allowed Paths`.
- Dependency changes are needed.
- Secrets are needed and not declared.
- Build requires unrelated fixes.
- Architecture differs from the linked guidelines.
- More than 3 unexpected files need edits.

## Review Rules

- A finished packet returns the standard report (Summary, Files Changed, Verification, Architecture Compliance, Risks / Open Questions, Suggested Wiki Updates).
- Suggested wiki updates go only to `inbox/hermes-proposals/`. Canonical edits require human promotion.
- A packet stuck in `running` or `review` for more than 14 days is reviewed during weekly maintenance (`meta/maintenance-checklist.md`).

## Dry-Run Order (Yawnly Pilot)

1. `research-only` on the first packet.
2. `plan-only` (if research uncovered a clear plannable change).
3. Human review of the report and the proposal.
4. Only after explicit approval, optionally `implement-local`.
5. `implement-pr` only after `implement-local` produced a clean diff and was reviewed.

## Naming

`yawnly-YYYYMMDD-<short-name>.md`. Short name uses kebab-case and is human-scannable.
