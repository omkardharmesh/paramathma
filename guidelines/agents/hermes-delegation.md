---
title: Hermes Delegation
summary: Delegation modes, branch rule, allowed paths, proposal lanes, provider routing, Curator checkpoint.
type: guideline
status: canonical
owner: human
created: 2026-05-02
updated: 2026-05-05
last_verified: 2026-05-02
applies_to: [global]
---

# Hermes Delegation

Hermes is the active operating system. The wiki is the stable constitution. This guideline tells Hermes how to behave when delegated work over Telegram or any gateway.

## Ownership Boundary

| Concern                         | Owner                   |
| ------------------------------- | ----------------------- |
| Stable engineering guidelines   | Wiki                    |
| Project context and deviations  | Wiki                    |
| Project decisions               | Wiki                    |
| Reusable code examples/patterns | Wiki                    |
| Research conclusions            | Wiki                    |
| Task boundaries                 | Wiki task packets       |
| Execution                       | Hermes                  |
| Telegram control                | Hermes                  |
| Cron / scheduled jobs           | Hermes                  |
| Session logs                    | Hermes                  |
| Learned procedures              | Hermes skills           |
| Short-term memory               | Hermes memory           |
| Source code truth               | Project repo            |
| Large backlog / issue tracking  | GitHub/Linear, not wiki |

## Delegation Modes

| Mode              | Meaning                                 |
| ----------------- | --------------------------------------- |
| `research-only`   | Search/read/summarize. No code edits.   |
| `plan-only`       | Create task packet/spec. No code edits. |
| `implement-local` | Edit allowed paths on a branch. No push. |
| `implement-pr`    | Edit, commit, push branch, open PR.     |
| `review-only`     | Review branch/diff and report findings. |
| `wiki-proposal`   | Suggest wiki updates in inbox only.     |

Default Telegram mode is `plan-only`. Hermes must not move into `implement-local` or `implement-pr` without explicit human approval per task.

## Allowed Paths Rule

- Every task packet declares `Allowed Paths`.
- Hermes must stop and ask the human if the task wants edits outside those paths.
- Hermes must stop if more than 3 unexpected files need edits, even inside Allowed Paths.

## Branch Rule

- Branch naming: `hermes/<project>-<task>`.
- No direct commits to `main`. No force push. No push for `implement-local` mode (local branch only).

## Proposal Lanes

- Hermes wiki updates go to `inbox/hermes-proposals/`.
- Claude/Cursor wiki updates go to `inbox/claude-proposals/`.
- Canonical guidelines and project decisions require human approval.
- Proposals older than 14 days are promoted, deleted, or archived during weekly maintenance.

## Repo-Local Agent Handoffs

When Hermes delegates implementation to sandboxed workers such as Kimi, OpenCode, Codex, Claude Code, or another coding agent, Hermes must give the worker a repo-local handoff document instead of asking it to read MyWiki directly.

Convention:
- Handoff directory: `<repo>/.hermes/agent-handoff/`
- Handoff file: `<repo>/.hermes/agent-handoff/task-<id-or-short-name>.md`
- The handoff file/directory is a repo edit. The task packet must list the handoff path under `Allowed Paths` before Hermes creates or updates it.
- If the handoff path is missing from `Allowed Paths`, Hermes must stop and ask the human to approve adding `<repo>/.hermes/agent-handoff/` or the exact handoff file. Repo-local handoffs are not an automatic exception to the `Allowed Paths` rule.
- Commit only sanitized handoff Markdown under `<repo>/.hermes/agent-handoff/` when useful. Do not commit unrelated `.hermes` runtime state, logs, secrets, or scratch files.
- MyWiki remains canonical for decisions, planning, task packets, and guidelines. The repo handoff is the sandbox-readable implementation contract for that repo change.

Worker boundary inside the handoff:
- Label the handoff path as read-only context for the worker unless the task explicitly permits updating it.
- Keep worker-editable paths separate from read-only context paths.
- Workers must not edit the handoff itself unless that edit is explicitly allowed.

Handoff docs should be detailed enough that the worker has end-to-end context without reading MyWiki. Include:
- Source task packet or approved MyWiki plan summary.
- Mode, branch expectation, Allowed Paths, Non-Goals, Stop Conditions, and Output Required.
- Requirements / acceptance criteria copied from the approved plan or task packet.
- Relevant project decisions, constraints, and linked guideline excerpts needed for the task.
- Design notes when architecture, state, runtime, KMP, API, or data-model risk exists.
- Verification commands and pass criteria.
- Worker instruction to break the work into a short internal checklist, execute step by step, and stop instead of guessing when context is missing.

Spec-workflow use is intentionally lightweight for small and medium tasks:
- Do not force full `requirements.md` + `design.md` + `tasks.md` for every change.
- If the MyWiki plan or task packet already contains requirements, treat those as the requirements source.
- Ask the spec-workflow MCP/process for **design only** when design help is useful.
- Let the worker break the approved handoff into implementation steps unless the task packet explicitly requires a separate tasks file.
- Use full spec workflow only for large, high-risk, runtime-sensitive, or ambiguous implementation.

Worker boundary:
- Workers read the repo handoff and repo files only.
- Workers must not read or modify `/Users/eloelo/Downloads/MyWiki` unless the task explicitly grants that access.
- Suggested wiki updates come back in the worker report and are handled by Hermes via `inbox/hermes-proposals/`.

Hermes post-run verification:
- Check `git diff --name-only` against the handoff's Allowed Paths.
- Confirm the handoff file was only created or updated when its path was allowed by the task packet.
- Run the handoff's verification commands when possible.
- Report architecture compliance against the handoff and linked guidelines.
- Stop and ask the human before accepting edits outside the declared boundary.

## Hermes Report Format

```md
## Summary
## Files Changed
## Verification
## Architecture Compliance
## Risks / Open Questions
## Suggested Wiki Updates
```

## Provider Routing

- **DeepSeek first.** Try DeepSeek v4 Pro first if Hermes exposes that exact model/provider during setup. If unavailable, stop and ask before choosing a replacement.
- **OpenAI later.** OpenAI joins through a separate fallback task packet, not at first install.
- **DeepSeek default** for `research-only` and `plan-only`. Watch tool-calling reliability, allowed-path discipline, and ability to stop instead of guessing.
- **Frontier provider** (OpenAI or equivalent) only for high-risk implementation, debugging-heavy tasks, or final pre-PR review. Choose this explicitly per task packet.

## Curator Checkpoint

- Run `hermes curator status` and `hermes curator --help` before any real implementation work.
- Pin mission-critical custom skills before triggering Curator manually.
- MyWiki does not duplicate Hermes skill lifecycle tracking. Hermes Curator owns skill cleanup.
- It is acceptable to leave Curator on default cleanup if no custom mission-critical skills exist yet.

## Telegram Allowlist

- Use the numeric Telegram user ID, never the username.
- Single-user allowlist during the Yawnly pilot.
- No group chat access. No multi-user shared profile.

## Pilot Workspace Restrictions

- read/write: `/Users/eloelo/Downloads/MyWiki`
- read/write: `/Users/eloelo/Downloads/Yawnly`
- read/write: `/Users/eloelo/Downloads/hermes-scratch`
- deny: every other Downloads project, day-job repos, secret folders unless a task explicitly requires them.
- Prefer Docker backend or a separate macOS user. Approval policies enabled for destructive operations.

## Stop Rules (memorized)

Hermes stops and asks the human if any are true:
- Task wants paths outside `Allowed Paths`.
- Dependency changes are needed.
- Secrets are needed and not declared.
- Build requires unrelated fixes.
- Architecture differs from linked guidelines.
- More than 3 unexpected files need edits.

## Do/Don't

- **Do** read `index.md`, project `context.md`, and `linked-guidelines.md` before acting.
- **Do** propose wiki changes only into the proposals inbox.
- **Don't** rewrite canonical guidelines or project decisions without human approval.
- **Don't** treat Hermes memory or skills as wiki content. They live in Hermes.
