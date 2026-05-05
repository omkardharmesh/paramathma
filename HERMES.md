---
title: HERMES.md
summary: Hermes-specific instructions, defaults, and recorded install state.
type: meta
status: canonical
updated: 2026-05-05
---

# HERMES.md — Hermes Operating Instructions

Read `AGENTS.md` first; this file adds Hermes-specific rules.

## Defaults

- Telegram commands default to `mode: plan-only`.
- Implementation requires an approved task packet.
- Branch naming: `hermes/<project>-<task>`.
- No push to `main`. No force push.
- Use `inbox/hermes-proposals/` for any wiki update suggestions.

## Agent Handoffs

For implementation delegated to sandboxed coding workers (OpenCode/Codex/etc.), create a durable repo-local handoff doc before launching the worker:

- Path: `<repo>/.hermes/agent-handoff/task-<id-or-short-name>.md`
- The handoff file/directory is a repo edit and must be listed in the task packet's `Allowed Paths` before Hermes creates or updates it. If the path is missing, stop and ask the human to approve adding it. Repo-local handoffs are not an automatic exception to the `Allowed Paths` rule.
- Commit only sanitized handoff Markdown under `<repo>/.hermes/agent-handoff/` when useful. Do not commit unrelated `.hermes` runtime state, logs, secrets, or scratch files.
- The handoff must be detailed enough that the worker can implement without reading MyWiki.
- The handoff should separate the worker's read-only context path from the worker-editable paths. Workers must not edit the handoff itself unless the task explicitly permits it.
- Include approved requirements from the MyWiki plan/task packet, Allowed Paths, Stop Conditions, relevant decisions/guidelines, verification commands, and output requirements.
- Use spec-workflow MCP/process in **design-only** mode when design help is needed. Do not force full requirements/design/tasks specs for small or medium work when the MyWiki plan already captures requirements.
- Worker agents should break the handoff into an internal checklist and execute step by step. If context is missing, they stop and report instead of guessing.
- Worker agents must not read or modify MyWiki unless the task explicitly grants that access. Suggested wiki updates return to Hermes and go through `inbox/hermes-proposals/`.

## Stop Conditions

Stop and ask the human if any of these are true:
- Task wants paths outside the task packet's `Allowed Paths`.
- Task needs dependency changes.
- Task needs secrets that are not declared.
- Build requires unrelated fixes.
- Architecture differs from the linked guidelines.
- More than 3 unexpected files need edits.

## Pilot Workspace Access

- read/write: `/Users/eloelo/Downloads/MyWiki`
- read/write: `/Users/eloelo/Downloads/Yawnly`
- read/write: `/Users/eloelo/Downloads/hermes-scratch`
- deny: every other Downloads project, day-job repos, secrets folders

## Provider Strategy

- Try DeepSeek v4 Pro first if Hermes exposes that model/provider during setup. If the exact model is unavailable, stop and ask before choosing a replacement.
- OpenAI is added later through a separate fallback task packet, not at first install.
- DeepSeek is the default for `research-only` and `plan-only`.
- A frontier provider (OpenAI or equivalent) is only chosen for high-risk implementation, debugging-heavy tasks, or when DeepSeek fails the dry run.

## Curator

- Run `hermes curator status` and `hermes curator --help` before any real implementation work.
- Pin mission-critical custom skills before triggering Curator manually.
- MyWiki does not duplicate Hermes skill lifecycle tracking — Curator owns skill cleanup.

## Telegram Allowlist

- Use the numeric Telegram user ID, never the username.
- Single-user allowlist during the pilot. No group chats. No multi-user shared profile.

## Recorded Install State

Filled in after Hermes install succeeds. Do not record API keys or tokens here.

- Hermes version: v0.12.0 (2026.4.30)
- Model/provider: deepseek-v4-pro via DeepSeek (base URL `https://api.deepseek.com`)
- Telegram gateway status: launchd service `ai.hermes.gateway` loaded; logs at `~/.hermes/logs/gateway.log`. Numeric user ID allowlisted, single-user.
- Approval policies enabled: defaults applied via `hermes setup` (max iterations 90, compression 0.50, daily reset 4:00, inactivity 1440 min)
- Workspace allowlist applied: pilot scope = `MyWiki`, `Yawnly`, `hermes-scratch`. Enforced per-task via task packet `Allowed Paths`.
- Curator status: ENABLED. Interval 7d. Runs: 0 (deferred first run). No manual run triggered.
- Pinned skills: none yet (no custom mission-critical skills exist).
- First Yawnly dry run: PASSED 2026-05-02 — research-only on `yawnly-20260502-stale-doc-reference-map.md`. Report: `inbox/hermes-proposals/yawnly-stale-doc-reference-map-20260502.md`. No Yawnly edits.

## First Dry Run (Yawnly Pilot)

Telegram message:

```
Yawnly: run research-only on /Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/yawnly-20260502-stale-doc-reference-map.md.
Do not edit source code. Return report only.
```

Pass conditions:
- Hermes reads MyWiki context.
- Hermes does not edit Yawnly source code.
- Hermes obeys the task packet's Allowed Paths.
- Hermes returns a useful report.
- Any wiki updates go only to `inbox/hermes-proposals/`.
