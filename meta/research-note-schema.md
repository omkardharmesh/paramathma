---
title: Research Note Schema
summary: Distillation format for research notes. Conclusions only, not dumps.
type: meta
status: canonical
updated: 2026-05-02
---

# Research Note Schema

Research is captured to produce 2-5 reusable points, not dumps. Notes start in `research/inbox/` or `inbox/quick-capture/`. They become canonical only after distillation into `research/accepted/`.

## Frontmatter

```yaml
---
title:
topic:
status: inbox | accepted | rejected
created: YYYY-MM-DD
projects: []
tags: []
---
```

## Required Sections

```md
# Research: <Topic>

## Question
What were we trying to answer?

## Keep
- Point worth remembering.
- Point worth remembering.

## Decision
What should we do?

## Rejected
- What did we reject and why?

## Applies To
- Global guideline?
- One project?
- Future project?

## Next Action
- Update guideline.
- Add project decision.
- Create task packet.
- Do nothing.

## Sources
- Link or source path
```

## Promotion Rule

A research note becomes canonical only when it has at least one entry under `Decision` and one under `Next Action`. If neither exists, the note stays in `inbox` or moves to `research/rejected/`.

## Size Cap

Accepted research notes should stay under 120 lines. If they grow past that, split into multiple notes or promote a stable conclusion into the matching guideline or project `decisions.md`.
