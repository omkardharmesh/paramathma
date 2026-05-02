---
title: Yawnly Bridge-Thinning — Implementation Plan
id: hermes-proposal-yawnly-bridge-thinning-plan-20260502
type: implementation-plan
project: yawnly
task_packet: yawnly-20260502-bridge-thinning
parent_packet: yawnly-20260502-stale-doc-reference-map
mode: plan-only
created: 2026-05-02
status: draft
---

# Yawnly Bridge-Thinning — Implementation Plan

**Plan-only deliverable.** Describes every line-level edit needed to thin Yawnly agent docs. No files edited — this is the review artifact before `implement-local`.

---

## Summary

- **Files in scope:** 6 (2 primary bridge files + 4 secondary cleanup files)
- **Edits proposed:** 8 auto-edits + 2 flagged for human decision
- **Lines that would change:** 11
- **Files preserved untouched:** All Yawnly-specific sections (build commands, architecture gotchas, UI conventions, review gates, feature layout, do-not-touch list, Supabase project details)
- **Verification:** Re-run 3 `rg` commands from the research report to confirm zero stale refs remain

---

## Per-File Edit Plan

### Edit 1 — `AGENTS.md` line 126

**Rationale:** `plan/yawnly-cmp-implementation-plan.md` does not exist. Replace with MyWiki project context + repo-map, which cover the same ground (stack, modules, feature packages, build commands, constraints, decisions).

**Before (line 126):**
```
| `plan/yawnly-cmp-implementation-plan.md` | Feature roadmap and phased build order |
```

**After:**
```
| `MyWiki/projects/yawnly/context.md` | Project context, stack, hard constraints, decisions |
| `MyWiki/projects/yawnly/repo-map.md` | Module layout, feature packages, Supabase backend, Edge Functions |
```

**Impact:** 1 line removed, 2 lines added. Table expands by 1 row. The existing 3 rows (agent-architecture.md, yawnly-ui-ux-polish-guide.md, CLAUDE.md) are untouched.

---

### Edit 2 — `CLAUDE.md` line 17 (Where Docs Live table)

**Rationale:** Same stale reference. Replace with MyWiki paths. Keep the `docs/llm-agent-feature-implementation-guide.md` migration note on line 16 intact — it's an acknowledged removal.

**Before (line 17):**
```
| **`plan/yawnly-cmp-implementation-plan.md`** | Concrete implementation map: feature paths, state contracts, route flow, and phased build order. |
```

**After:**
```
| **`MyWiki/projects/yawnly/context.md`** | Stable project context: stack, feature packages, hard constraints, decisions, Supabase backend. |
| **`MyWiki/projects/yawnly/repo-map.md`** | Module layout, source tree, Edge Functions list, in-repo doc index, stale references. |
```

**Impact:** 1 line replaced, 1 new row added. Table grows by 1.

---

### Edit 3 — `CLAUDE.md` lines 21–26 (Read Order → First Read Protocol)

**Rationale:** This is the most critical edit. The "Read These First" block is the agent boot sequence. Step 1 points at a missing file. Replace with a First Read Protocol that mirrors MyWiki's protocol, keeping in-repo docs as secondary reads.

**Before (lines 21–26):**
```
## Read These First (in order)

1. **`plan/yawnly-cmp-implementation-plan.md`** — Implementation plan: feature structures, file paths, state, routes, build order.
2. **`docs/agent-architecture.md`** — Architecture reference (layers, DI, anti-patterns).
3. **`docs/yawnly-ui-ux-polish-guide.md`** — UI/UX polish; use with step 2 and **Compose UI Rules** below.
4. **`docs/codebase-review.md`** — Optional inventory/risk snapshot for audits and repository orientation.
```

**After:**
```
## First Read Protocol

1. Read **`MyWiki/index.md`** — canonical entry point and First Read Protocol.
2. Read **`MyWiki/projects/yawnly/context.md`** — project overview, stack, hard constraints, decisions D001–D008.
3. Read **`MyWiki/projects/yawnly/linked-guidelines.md`** — which global guidelines apply to this task.
4. Read only the linked guideline pages required by the current task.
5. Read the current **task packet** (if working from one).
6. Then read **`docs/agent-architecture.md`** — Yawnly-specific architecture: layers, DI, navigation, anti-patterns.
7. Read **`docs/yawnly-ui-ux-polish-guide.md`** — UI tokens, animations, spacing conventions.

Do not recursively load the wiki. See `MyWiki/meta/wikiignore.md`.

**`docs/codebase-review.md`** remains available as an optional inventory/risk snapshot for audits.

The build commands, architecture gotchas, UI conventions, review gates, and feature layout below are Yawnly-specific deltas — follow them after loading the wiki context.
```

**Verification:** The 4 original documents are all still referenced (agent-architecture.md, yawnly-ui-ux-polish-guide.md, codebase-review.md, CLAUDE.md), just reordered and contextualized within the wiki-first protocol.

---

### Edit 4 — `docs/agent-architecture.md` line 5 (Pair With)

**Rationale:** Replace stale impl-plan ref with wiki project context.

**Before (line 5):**
```
**Pair with:** `docs/yawnly-ui-ux-polish-guide.md` (Yawnly UI polish), `CLAUDE.md` (product + non-negotiables), `plan/yawnly-cmp-implementation-plan.md` (roadmap + route/feature map).
```

**After:**
```
**Pair with:** `docs/yawnly-ui-ux-polish-guide.md` (Yawnly UI polish), `CLAUDE.md` (product + non-negotiables), `MyWiki/projects/yawnly/context.md` (project context + decisions D001–D008).
```

**Impact:** 1 line changed. docs/yawnly-ui-ux-polish-guide.md and CLAUDE.md references preserved.

---

### Edit 5 — `docs/agent-architecture.md` line 364 (Document Map table)

**Rationale:** Same stale reference. Replace with wiki path.

**Before (line 364):**
```
| **`plan/yawnly-cmp-implementation-plan.md`** | Routes, features, phased plan. |
```

**After:**
```
| **`MyWiki/projects/yawnly/context.md`** | Project context, stack, hard constraints, decisions. |
```

**Impact:** 1 line replaced.

---

### Edit 6 — `docs/agent-architecture.md` line 365 (Document Map table)

**Rationale:** `plan/yawnly-prd-v3.md` does not exist and the line itself says "v4 in repo for current PRD" — but v4 doesn't exist either. Replace with CLAUDE.md, which contains the product summary.

**Before (line 365):**
```
| **`plan/yawnly-prd-v3.md`** | Product requirements (v4 in repo for current PRD). |
```

**After:**
```
| **`CLAUDE.md`** | Product summary and requirements. |
```

**Impact:** 1 line replaced.

---

### Edit 7 — `plan/v2/pending/release-pending.md` line 5 (See Also)

**Rationale:** Both `plan/yawnly-prd-v4.md` and `plan/yawnly-cmp-implementation-plan.md` are missing. Keep `CLAUDE.md` which exists, add wiki ref.

**Before (line 5):**
```
See also: `plan/yawnly-prd-v4.md`, `plan/yawnly-cmp-implementation-plan.md`, `CLAUDE.md`.
```

**After:**
```
See also: `CLAUDE.md`, `MyWiki/projects/yawnly/context.md`.
```

**Impact:** 1 line changed. 3 refs → 2 refs, both extant.

---

### Edit 8 — `plan/v2/pending/release-pending.md` line 67 (Story Quality Task)

**Rationale:** `plan/release-v1.1-findings.md` does not exist and no release findings doc of any version exists in the repo. Replace with self-referential note.

**Before (line 67, excerpt):**
```
—**without sending PII to the LLM** (see placeholder rule below; align with `plan/release-v1.1-findings.md` where applicable).
```

**After:**
```
—**without sending PII to the LLM** (see placeholder rule below).
```

**Impact:** 1 inline phrase removed. The sentence is complete without it. The placeholder rule referenced is in the same doc's bullet points below.

---

## Human-Input-Needed (2 findings)

These findings require a human decision before editing. They are flagged here but excluded from the auto-edit scope.

### H1 — `plan/v2/komodo/komodo_integration_impl_spec.md` line 41

**Context:**
```
The two reference docs (`komodo-app-integration-detailed.md` and `komodo_integration_spec.md`) describe a **complete 3-tier billing architecture** from another project that Yawnly must replicate:
```

**Issue:** `komodo-app-integration-detailed.md` does not exist in the Yawnly repo (or the komodo directory). All komodo specs are marked IMPLEMENTED. This is a cross-project reference.

**Options:**
| Option | Edit |
|---|---|
| A — Annotate as external | `The reference doc (`komodo_integration_spec.md`) and an external spec from another project describe...` |
| B — Delete the reference | `The reference doc (`komodo_integration_spec.md`) describes a **complete 3-tier billing architecture**...` |
| C — Vendor the missing doc | Copy `komodo-app-integration-detailed.md` into the komodo directory (requires finding it first) |

**Recommendation: Option A** — the komodo specs are historical/archival; annotating preserves context without pretending the file exists.

---

### H2 — `modal/impl.md` line 10

**Context:**
```
Full Modal deployment scripts and voice configs are finalized in `/Users/eloelo/Downloads/Modal/TTS_INTEGRATION_DOC.md`.
```

**Issue:** The referenced file exists but is **outside** the Yawnly repo. If Modal is an active sibling project, this is fine. If Modal is abandoned or the doc is irrelevant, the reference is noise.

**Options:**
| Option | Edit |
|---|---|
| A — Keep with annotation | `...finalized in \`/Users/eloelo/Downloads/Modal/TTS_INTEGRATION_DOC.md\` (external sibling project).` |
| B — Remove | Replace with: `...finalized externally (contact the Modal project for details).` |
| C — Copy inline | Summarize the relevant TTS config in `modal/impl.md` (likely out of scope for bridge-thinning) |

**Recommendation: Option A** — the file exists and the reference is intentional (Modal is the TTS backend). The path is absolute and clear. Just annotate.

---

## Preserved Sections (verification that Yawnly-specific knowledge survives)

These sections receive **zero edits** in this plan:

| File | Section | Lines | Content preserved |
|---|---|---|---|
| `AGENTS.md` | Mandatory Skill Invocation | 1–5 | SKILL.md loading rule |
| `AGENTS.md` | Planning Protocol | 5–15 | GO-before-code rule |
| `AGENTS.md` | Build & Test | 15–25 | `rtk ./gradlew` commands |
| `AGENTS.md` | Architecture Gotchas | 25–40 | CMP library vs app, two stacks, Koin both platforms |
| `AGENTS.md` | Review Gates | 40–50 | Layer contract, ApiService transport-only |
| `AGENTS.md` | UI Conventions | 50–60 | SDP/SSP, UniversalColor, BaseActionButton, noRippleSafeClick |
| `AGENTS.md` | Icon Convention | 60–65 | icon-fetch skill |
| `AGENTS.md` | Feature File Order | 65–100 | Full directory tree template |
| `AGENTS.md` | New Feature Checklist | 100–110 | 9-step checklist |
| `AGENTS.md` | Execution Conventions | 128–135 | `rtk` prefix, no Room, analytics post-MVP |
| `AGENTS.md` | Supabase DB Query Convention | 135–144 | `--linked` vs `--local` queries |
| `CLAUDE.md` | Do NOT Touch | 28–33 | core/, BaseRepository, BaseViewModel, theme |
| `CLAUDE.md` | Key Architecture Rules | 35–41 | Package, clean arch, SupabaseBaseRepository |
| `CLAUDE.md` | Supabase Project | 42–47 | Project ID, URL, keys location |

---

## Verification Plan

After `implement-local`, re-run these commands and confirm zero stale refs:

```bash
# 1. Confirm impl-plan references are gone (except in research artifacts)
rg -n "yawnly-cmp-implementation-plan" /Users/eloelo/Downloads/Yawnly \
  --glob "*.md" --glob '!**/graphify-out/**' --glob '!**/build/**'

# 2. Confirm PRD references are gone
rg -n "yawnly-prd" /Users/eloelo/Downloads/Yawnly \
  --glob "*.md" --glob '!**/graphify-out/**' --glob '!**/build/**'

# 3. Confirm release-v1.1-findings reference is gone
rg -n "release-v1.1-findings" /Users/eloelo/Downloads/Yawnly \
  --glob "*.md" --glob '!**/graphify-out/**' --glob '!**/build/**'

# 4. Confirm new MyWiki references resolve (read the files)
rg -n "MyWiki/" /Users/eloelo/Downloads/Yawnly \
  --glob "*.md" --glob '!**/graphify-out/**' --glob '!**/build/**'
```

Expected results:
- Command 1: 0 matches (or only in `plan/v2/pending/release-pending.md` if we intentionally keep the historical see-also — but edit 7 removes it)
- Command 2: 0 matches
- Command 3: 0 matches
- Command 4: 7+ matches (all pointing to extant wiki pages)

---

## Risks / Open Questions

1. **CLAUDE.md Read Order replacement is the highest-risk edit.** If the new "First Read Protocol" block is less clear than the original, agents may skip context. The replacement is modeled directly on MyWiki's own protocol (index.md lines 13–21), which is battle-tested.
2. **MyWiki paths use `MyWiki/` prefix.** These are absolute from the user's machine (`/Users/eloelo/Downloads/MyWiki/`). If MyWiki moves, all bridge refs become stale. This is an accepted risk — absolute paths are more explicit than relative `../MyWiki/` paths.
3. **`docs/agent-architecture.md` is used by non-Hermes agents (Claude, Cursor).** Edits 4–6 change its document map. The replacements preserve all functional doc relationships — no capability is lost for non-Hermes agents.
4. **Komodo and Modal references remain unresolved.** Edits H1–H2 are intentionally deferred to human. If not resolved before `implement-local`, they will persist as stale refs in secondary plan files.
5. **The `plan/v2/pending/release-pending.md` line 67 edit removes "align with `plan/release-v1.1-findings.md` where applicable".** If that findings doc contained meaningful content, it may be lost. However, no such file exists and it was never created — the risk is theoretical.

---

## Files Changed (would be, in implement-local)

| File | Lines changed | Type |
|---|---|---|
| `Yawnly/AGENTS.md` | 126 (replaced, 1 row → 2 rows) | Stale ref removal + wiki pointers |
| `Yawnly/CLAUDE.md` | 14–26 (table row replaced + Read Order rewritten) | Boot sequence rewrite |
| `Yawnly/docs/agent-architecture.md` | 5, 364, 365 | Stale ref removal + wiki pointers |
| `Yawnly/plan/v2/pending/release-pending.md` | 5, 67 | Stale ref removal |
| `Yawnly/plan/v2/komodo/komodo_integration_impl_spec.md` | 41 | **Deferred (H1)** |
| `Yawnly/modal/impl.md` | 10 | **Deferred (H2)** |

**Total auto-edits: 8** across 4 files (11 lines changed).  
**Deferred to human: 2** across 2 files.

---

## Architecture Compliance

Against `guidelines/agents/hermes-delegation.md`:
- ✅ Plan-only mode — no code edits, no Yawnly file writes
- ✅ Allowed Paths respected — reads from Yawnly + MyWiki, writes only to `inbox/hermes-proposals/`
- ✅ Stop Conditions checked — no dependency changes, no secrets, no source code edits, no unexpected files
- ✅ Non-Goals respected — no architecture changes, no new Yawnly docs, no canonical wiki edits

---

## Suggested Wiki Updates

None at this stage. The plan itself is sufficient for human review. After `implement-local` completes, a `wiki-proposal` packet could update `projects/yawnly/current-state.md` to reflect that bridge-thinning is done (moving it from "candidates" to "completed").
