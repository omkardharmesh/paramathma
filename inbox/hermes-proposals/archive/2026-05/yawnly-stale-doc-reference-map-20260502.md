---
title: Yawnly Stale Doc Reference Map
id: hermes-proposal-yawnly-stale-doc-refs-20260502
type: research-report
project: yawnly
task_packet: yawnly-20260502-stale-doc-reference-map
mode: research-only
created: 2026-05-02
status: draft
---

# Yawnly — Stale Doc Reference Map

**Research-only report.** No source code or wiki edits. Produced per `projects/yawnly/task-packets/yawnly-20260502-stale-doc-reference-map.md`.

---

## Summary

- **Citing files scanned:** 10 markdown files with active doc references
- **Unique cited paths checked:** 16
- **Stale/missing references:** 6 (across 4 citing files)
- **Outside-repo reference:** 1
- **Superseded-but-acknowledged:** 1 (not actionable)
- **Recommendation:** Bridge-thinning — point `AGENTS.md`/`CLAUDE.md` at MyWiki, prune stale refs

---

## Findings

### Finding 1: `plan/yawnly-cmp-implementation-plan.md` — MISSING (CRITICAL)

**Status:** File does not exist in the repo.

This is the highest-impact stale reference. It is cited as a *primary architectural document* in the agent boot sequence and Key Docs tables.

| Citing File | Line | Context |
|---|---|---|
| `AGENTS.md` | 126 | Key Docs table: "Feature roadmap and phased build order" |
| `CLAUDE.md` | 17 | Key Docs table: "Concrete implementation map: feature paths, state contracts, route flow, and phased build order" |
| `CLAUDE.md` | 23 | Read Order step 1 — **first doc agents must read** |
| `docs/agent-architecture.md` | 5 | Header preamble: "Pair with..." |
| `docs/agent-architecture.md` | 364 | Key Docs table: "Routes, features, phased plan" |
| `plan/v2/pending/release-pending.md` | 5 | "See also" line |

**Recommendation:** **Delete the citation.** Point `AGENTS.md` and `CLAUDE.md` at MyWiki (`projects/yawnly/context.md` + `projects/yawnly/repo-map.md`) for stable project truths. If phased implementation order is actually needed, re-create it as a wiki page — but current state shows all 8 feature scaffolds exist and no active implementation phase is underway (Hermes pilot is still gated).

---

### Finding 2: `plan/yawnly-prd-v4.md` — MISSING

**Status:** File does not exist. No PRD file of any version exists in the repo (`plan/yawnly-prd-v*` search returned zero results).

| Citing File | Line | Context |
|---|---|---|
| `plan/v2/pending/release-pending.md` | 5 | "See also: `plan/yawnly-prd-v4.md`, `plan/yawnly-cmp-implementation-plan.md`, `CLAUDE.md`" |

**Recommendation:** **Delete the citation.** Product requirements are already spread across `CLAUDE.md` (product summary + do-not-touch list), `docs/agent-architecture.md` (architecture), `plan/v2/pending/release-pending.md` (release queue), and MyWiki `projects/yawnly/context.md`. A standalone PRD adds maintenance burden without benefit at this stage.

---

### Finding 3: `plan/yawnly-prd-v3.md` — MISSING

**Status:** File does not exist.

| Citing File | Line | Context |
|---|---|---|
| `docs/agent-architecture.md` | 365 | Key Docs table: "Product requirements (v4 in repo for current PRD)" |

**Recommendation:** **Replace** with `CLAUDE.md` or `projects/yawnly/context.md`. The line itself acknowledges v4 as current, making the v3 citation self-defeating. A single canonical pointer is better than versioned PRD paths that rot.

---

### Finding 4: `plan/release-v1.1-findings.md` — MISSING

**Status:** File does not exist. Only `plan/v2/pending/release-pending.md` exists in the release directory.

| Citing File | Line | Context |
|---|---|---|
| `plan/v2/pending/release-pending.md` | 67 | Story quality task: "align with `plan/release-v1.1-findings.md` where applicable" |

**Recommendation:** **Delete the citation** or replace with a note that findings are folded into the current release-pending doc. The v1.1 findings doc was apparently never created or has been absorbed.

---

### Finding 5: `plan/v2/komodo/komodo-app-integration-detailed.md` — MISSING

**Status:** File does not exist. The komodo directory contains only 4 files: `komodo_integration_impl_spec.md`, `komodo_integration_spec.md`, `supabase_iap_server_impl_spec.md`, and `komodo_supabase_wiring_map.md`.

| Citing File | Line | Context |
|---|---|---|
| `plan/v2/komodo/komodo_integration_impl_spec.md` | 41 | "The two reference docs (`komodo-app-integration-detailed.md` and `komodo_integration_spec.md`) describe a complete 3-tier billing architecture from another project" |

**Recommendation:** **Hold for human input.** All komodo specs are marked IMPLEMENTED. This is a cross-project reference to another app's billing architecture — it may have been intentionally excluded from the Yawnly repo. If the reference is purely historical, delete or annotate. If the doc should be vendored, copy it in.

---

### Finding 6: `docs/llm-agent-feature-implementation-guide.md` — REMOVED (ACKNOWLEDGED)

**Status:** File was intentionally removed; content merged into `docs/agent-architecture.md`. Both citations explicitly acknowledge this migration.

| Citing File | Line | Context |
|---|---|---|
| `CLAUDE.md` | 16 | "Supersedes the removed `docs/llm-agent-feature-implementation-guide.md` (migration called out in that doc's header)" |
| `docs/agent-architecture.md` | 7 | "The old `docs/llm-agent-feature-implementation-guide.md` was removed; its useful content lives in this file. Update any bookmarks or rules that still cite that path." |

**Recommendation:** **No action.** These are migration notices, not stale references. Keep as-is.

---

### Outside-Repo Reference

| Citing File | Line | Referenced Path | Status |
|---|---|---|---|
| `modal/impl.md` | 10 | `/Users/eloelo/Downloads/Modal/TTS_INTEGRATION_DOC.md` | File exists but is **outside** the Yawnly repo |

**Recommendation:** **Hold for human input.** This is a cross-project reference to a Modal TTS integration doc. Acceptable if Modal is an active sibling project. If Modal is abandoned or the doc is irrelevant, delete the citation.

---

## Grouped by Citing File

### `AGENTS.md` (2 findings)
| Finding | Line | Target | State |
|---|---|---|---|
| #1 | 126 | `plan/yawnly-cmp-implementation-plan.md` | MISSING |

*Also has a commented-out reference to `graphify-out/GRAPH_REPORT.md` at line 113 — not active, no action.*

### `CLAUDE.md` (2 findings)
| Finding | Line | Target | State |
|---|---|---|---|
| #1 | 17, 23 | `plan/yawnly-cmp-implementation-plan.md` | MISSING |
| #6 | 16 | `docs/llm-agent-feature-implementation-guide.md` | ACKNOWLEDGED (no action) |

### `docs/agent-architecture.md` (2 findings)
| Finding | Line | Target | State |
|---|---|---|---|
| #1 | 5, 364 | `plan/yawnly-cmp-implementation-plan.md` | MISSING |
| #3 | 365 | `plan/yawnly-prd-v3.md` | MISSING |
| #6 | 7 | `docs/llm-agent-feature-implementation-guide.md` | ACKNOWLEDGED (no action) |

### `plan/v2/pending/release-pending.md` (3 findings)
| Finding | Line | Target | State |
|---|---|---|---|
| #1 | 5 | `plan/yawnly-cmp-implementation-plan.md` | MISSING |
| #2 | 5 | `plan/yawnly-prd-v4.md` | MISSING |
| #4 | 67 | `plan/release-v1.1-findings.md` | MISSING |

### `plan/v2/komodo/komodo_integration_impl_spec.md` (1 finding)
| Finding | Line | Target | State |
|---|---|---|---|
| #5 | 41 | `komodo-app-integration-detailed.md` | MISSING |

### `modal/impl.md` (1 finding)
| Finding | Line | Target | State |
|---|---|---|---|
| — | 10 | `/Users/eloelo/Downloads/Modal/TTS_INTEGRATION_DOC.md` | Outside repo |

---

## Intact References (verified)

These referenced files exist and are current — no action needed:

| Referenced File | Cited By |
|---|---|
| `docs/agent-architecture.md` | AGENTS.md, CLAUDE.md, README.md, plan/v2/pending/release-pending.md, self |
| `docs/yawnly-ui-ux-polish-guide.md` | AGENTS.md, CLAUDE.md, docs/agent-architecture.md |
| `docs/codebase-review.md` | CLAUDE.md, docs/agent-architecture.md |
| `CLAUDE.md` | AGENTS.md, docs/agent-architecture.md, plan/v2/pending/release-pending.md, docs/codebase-review.md |
| `skills/future-development/SKILL.md` | AGENTS.md |
| `plan/v2/komodo/komodo_supabase_wiring_map.md` | komodo_integration_impl_spec.md, komodo_integration_spec.md, supabase_iap_server_impl_spec.md |
| `plan/v2/komodo/komodo_integration_spec.md` | komodo_integration_impl_spec.md |
| `graphify-out/GRAPH_REPORT.md` | AGENTS.md (commented), skills/future-development/SKILL.md (commented) |

---

## Recommendation: Bridge-Thinning

The findings strongly support **bridge-thinning** over re-creating the missing implementation plan. Rationale:

1. **The dominant stale reference** (`plan/yawnly-cmp-implementation-plan.md`) is cited as a primary architectural doc, but its content is now distributed across MyWiki (`projects/yawnly/context.md`, `repo-map.md`, `decisions.md`) and in-repo docs (`docs/agent-architecture.md`). Re-creating it would add a fourth source of truth.
2. **No active implementation phase exists.** All 8 feature scaffolds are in place. The Hermes pilot hasn't started. There is no phased build to document.
3. **The wiki already covers what the plan would contain** — stack, modules, feature packages, build commands, layout constraints, and decisions D001–D008.
4. **Bridge-thinning is the natural next step in the pilot sequence** (`current-state.md` names it as candidate task packet #2).

### Proposed Follow-Up Packet

A `plan-only` packet: `task-packets/yawnly-YYYYMMDD-bridge-thinning.md`

Goal: Thin `Yawnly/AGENTS.md` and `Yawnly/CLAUDE.md` so they point to MyWiki for stable truths and keep only Yawnly-specific deltas. Specific edits:

1. Replace `plan/yawnly-cmp-implementation-plan.md` references with `MyWiki/projects/yawnly/context.md` + `MyWiki/projects/yawnly/repo-map.md`
2. Remove `plan/yawnly-prd-v4.md` and `plan/yawnly-prd-v3.md` citations
3. Remove `plan/release-v1.1-findings.md` citation
4. Add a "First Read Protocol" block that mirrors MyWiki's protocol
5. Preserve Yawnly-specific content: build commands, architecture gotchas, UI conventions, feature file layout, review gates

At least **2 files edited** (AGENTS.md, CLAUDE.md), **0 new files created**, **0 source code touched**.

---

## Risks / Open Questions

1. **`plan/yawnly-cmp-implementation-plan.md` references in CLAUDE.md line 23 are in the Read Order section.** Removing them changes the agent boot sequence. The replacement must be equally clear so agents don't skip context.
2. **`komodo-app-integration-detailed.md`** — unclear whether it should exist. It's cited as a cross-project reference. Human should decide: delete, vend, or annotate.
3. **`/Users/eloelo/Downloads/Modal/TTS_INTEGRATION_DOC.md`** — outside-repo reference. Is Modal an active project? If Modal is abandoned, this cross-ref is noise.
4. **No PRD exists.** Product requirements are distributed across CLAUDE.md, context.md, release-pending.md, and agent-architecture.md. Is this acceptable, or does a consolidated PRD have value?
5. **All komodo specs are marked IMPLEMENTED.** Are they still actively maintained, or should they be marked archival?

---

## Non-Findings (verified clean)

- All `docs/` references in AGENTS.md and CLAUDE.md resolve to existing files
- All komodo directory cross-references (except `komodo-app-integration-detailed.md`) resolve correctly
- The `skills/future-development/SKILL.md` path exists
- No secrets were encountered in any scanned file
- Reference count stayed well under the 200-finding threshold
- Generated artifacts (`graphify-out/`, `build/`, `.gradle/`) were excluded

---

## Verification

Commands run and results:

```
rg -n "\[.*\]\(([^)]+\.md)\)" ... → 1 markdown-style link found (README.md → docs/agent-architecture.md)
rg -n "`([^`]+\.md)`" ...           → 37 backtick references extracted (deduplicated to 16 unique paths)
rg -n "yawnly-cmp-implementation-plan|yawnly-prd|llm-agent-feature-implementation-guide" ... → 9 hits (all captured above)
find ... -name "*prd*.md"           → 0 results (confirmed no PRD files exist)
find ... -name "*release*.md"       → 1 result (release-pending.md only)
test -f for each of 16 unique paths → 10 exist, 5 missing, 1 outside repo
```

---

## Suggested Wiki Updates

None — this is a research-only report with no canonical wiki edits. All proposals stay in `inbox/hermes-proposals/`.
