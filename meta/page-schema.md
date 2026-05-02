---
title: Page Schema
summary: Frontmatter contract for canonical and project pages.
type: meta
status: canonical
updated: 2026-05-02
---

# Page Schema

Every canonical wiki page uses this frontmatter:

```yaml
---
title:
summary:
type: guideline | project-context | decision-log | current-state | repo-map | research | task-packet | meta
status: canonical | draft | stale | archived
owner: human | hermes-proposal | claude-proposal
created: YYYY-MM-DD
updated: YYYY-MM-DD
last_verified: YYYY-MM-DD
applies_to: []
supersedes:
---
```

Project files may use a minimal schema:

```yaml
---
title:
summary:
type:
status:
updated:
---
```

## Field Notes

- `summary` — one-line scan target. Agents may read summaries cheaply before opening a page.
- `last_verified` — set when a human or approved agent run confirmed the content is still true. Do not update on every read.
- `applies_to` — list of project slugs (e.g. `[yawnly]`) or `[global]`.
- `supersedes` — file path of the previous canonical page this one replaces.

## Invariants

- Page frontmatter MUST NOT be modified solely to record that an agent read the page.
- Read tracking, if ever needed, lives in `meta/page-usage.tsv` (not created at launch).
- A `status: stale` or `status: archived` page is not part of the active read protocol and MUST NOT be cited as canonical.

## Size Caps

| File type              | Soft cap  |
| ---------------------- | --------- |
| `index.md`             | 150 lines |
| guideline              | 300 lines |
| project `context.md`   | 150 lines |
| project `decisions.md` | 300 lines |
| `current-state.md`     | 100 lines |
| task packet            | 150 lines |
| accepted research note | 120 lines |

Enforcement is manual at launch. A size-check script is added later only if drift appears.
