---
title: Wiki Ignore
summary: Paths agents must not bulk-load.
type: meta
status: canonical
updated: 2026-05-02
---

# Wiki Ignore

Agents must not bulk-load:

- `research/rejected/`
- `inbox/`
- `golden-examples/`
- generated reports copied into project folders (`graphify-out/`, `*-report.md` snapshots, etc.)
- archived or stale pages (`status: archived`, `status: stale`)
- task packets that are not linked from the current project's `task-packets/README.md` or `current-state.md`
- `llm-wiki-plan-v3.md` at the wiki root — it is canonical strategy/seed input, not runtime context. Read only when reasoning about wiki structure itself.

Agents may read ignored paths only when a task packet explicitly lists the path under `Required Context` or `Allowed Paths`.

## Read Order Reminder

1. `index.md`
2. `projects/<project>/context.md`
3. `projects/<project>/linked-guidelines.md`
4. only the linked guideline and golden-example pages relevant to the task
5. the current task packet

`index.md` is the map, not the manual.
