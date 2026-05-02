---
title: Hermes Delegation
summary: Delegation modes, branch rule, allowed paths, proposal lanes, provider routing, Curator checkpoint.
type: guideline
status: canonical
owner: human
created: 2026-05-02
updated: 2026-05-02
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
