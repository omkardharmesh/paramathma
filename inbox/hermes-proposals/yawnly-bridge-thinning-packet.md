---
id: yawnly-20260502-bridge-thinning
project: yawnly
repo_path: /Users/eloelo/Downloads/Yawnly
status: draft
mode: plan-only
risk: medium
created: 2026-05-02
owner: human
executor: hermes
parent_packet: yawnly-20260502-stale-doc-reference-map
research_report: inbox/hermes-proposals/yawnly-stale-doc-reference-map-20260502.md
---

# Task: Bridge Thinning — Point Yawnly Agent Docs at MyWiki

## Goal

Thin `Yawnly/AGENTS.md` and `Yawnly/CLAUDE.md` so they point to MyWiki (`projects/yawnly/`) for stable truths and keep only Yawnly-specific deltas that the wiki intentionally does not duplicate. Also clean the 4 secondary files (`docs/agent-architecture.md`, `plan/v2/pending/release-pending.md`, `plan/v2/komodo/komodo_integration_impl_spec.md`, `modal/impl.md`) of stale or broken doc references.

## Why

The stale-doc research (`yawnly-20260502-stale-doc-reference-map`) found 6 missing references across 6 files. The dominant stale reference — `plan/yawnly-cmp-implementation-plan.md` — is cited as a *primary architectural document* in the agent boot sequence but does not exist. If Hermes (or any coding agent) trusts these stale references, it works from false context. The pilot must start from clean documentation.

The wiki already covers what the plan would contain: stack, modules, feature packages, build commands, architecture constraints, and decisions D001–D008. Duplicating it in-repo creates a maintenance fork. Bridge-thinning consolidates truth in one place.

## End Goal

A `plan-only` report delivered to `inbox/hermes-proposals/` that:
1. Lists the exact line-level edits needed in each of the 6 affected files.
2. Provides before/after snippets for each edit.
3. Validates that no Yawnly-specific knowledge is lost (build commands, architecture gotchas, UI conventions, review gates, feature layout all stay intact).
4. Identifies which findings need human input before editing (komodo cross-ref, Modal outside-repo ref).
5. Proposes a verification plan: re-run the reference-extraction commands from the research report and confirm zero stale refs remain.
6. Is ready for human review before promotion to `implement-local`.

## Output Format

A single markdown file at `inbox/hermes-proposals/yawnly-bridge-thinning-plan-20260502.md` containing:

- Per-file edit table (file, line, old text, new text, rationale)
- Before/after snippets for the 3 highest-impact edits (CLAUDE.md read order, AGENTS.md Key Docs table, agent-architecture.md header)
- Human-input-needed section (komodo, Modal)
- Verification commands to run after implementation
- Count of files touched and lines changed

No source code edits. No canonical wiki edits. No Yawnly file writes.

## Recommended Approach

### Phase 1: Load context
1. Re-read the research report at `inbox/hermes-proposals/yawnly-stale-doc-reference-map-20260502.md`.
2. Read the target sections of all 6 affected files to capture exact before-text:
   - `Yawnly/AGENTS.md` lines 120–127 (Key Docs table)
   - `Yawnly/CLAUDE.md` lines 1–27 (full header, where docs live, read order)
   - `Yawnly/docs/agent-architecture.md` lines 1–8 (header/pairing) and 358–365 (Key Docs table)
   - `Yawnly/plan/v2/pending/release-pending.md` lines 1–6 (header/see-also) and 64–70 (story quality section)
   - `Yawnly/plan/v2/komodo/komodo_integration_impl_spec.md` lines 38–44 (reference docs mention)
   - `Yawnly/modal/impl.md` lines 8–12 (Modal TTS doc reference)
3. Read `MyWiki/projects/yawnly/context.md` to confirm which wiki paths to cite.
4. Read `MyWiki/index.md` sections 13–21 (First Read Protocol) as the model for the new boot sequence.

### Phase 2: Draft edits

For each file, produce a precise edit with old text and new text.

#### Primary targets — bridge files (AGENTS.md, CLAUDE.md)

| # | File | Line(s) | Edit |
|---|---|---|---|
| 1 | `AGENTS.md` | 126 | Replace `plan/yawnly-cmp-implementation-plan.md` row with MyWiki project context ref |
| 2 | `CLAUDE.md` | 14–19 | Replace `plan/yawnly-cmp-implementation-plan.md` row with MyWiki refs. Keep `docs/agent-architecture.md`, `docs/yawnly-ui-ux-polish-guide.md`, `docs/codebase-review.md` rows. Keep line 16's note about `llm-agent-feature-implementation-guide.md` removal. |
| 3 | `CLAUDE.md` | 21–26 | Replace "Read These First (in order)" block with a "First Read Protocol" that mirrors MyWiki's 5-step protocol, pointing at MyWiki paths first, then in-repo docs |

#### Secondary targets — other stale refs

| # | File | Line(s) | Edit |
|---|---|---|---|
| 4 | `docs/agent-architecture.md` | 5 | Remove `plan/yawnly-cmp-implementation-plan.md` from the "Pair with" line. Replace with wiki ref. |
| 5 | `docs/agent-architecture.md` | 364 | Remove `plan/yawnly-cmp-implementation-plan.md` row from Key Docs table |
| 6 | `docs/agent-architecture.md` | 365 | Replace `plan/yawnly-prd-v3.md` row with wiki ref or delete (v3 is dead, v4 doesn't exist) |
| 7 | `plan/v2/pending/release-pending.md` | 5 | Replace "See also" line: remove `plan/yawnly-prd-v4.md` and `plan/yawnly-cmp-implementation-plan.md`, keep `CLAUDE.md` |
| 8 | `plan/v2/pending/release-pending.md` | 67 | Remove `plan/release-v1.1-findings.md` reference. Replace with note that findings are folded into this doc. |

#### Human-input-needed — do not auto-edit

| # | File | Line | Finding | Action |
|---|---|---|---|---|
| 9 | `plan/v2/komodo/komodo_integration_impl_spec.md` | 41 | `komodo-app-integration-detailed.md` missing | Flag. If cross-project ref is intentional, annotate with `[external]`. If stale, delete. |
| 10 | `modal/impl.md` | 10 | `/Users/eloelo/Downloads/Modal/TTS_INTEGRATION_DOC.md` outside repo | Flag. If Modal is active, annotate. If abandoned, delete. |

### Phase 3: Validate preservation

Verify these Yawnly-specific sections are **untouched**:
- `AGENTS.md`: Mandatory Skill Invocation, Planning Protocol, Build & Test, Architecture Gotchas, Review Gates, UI Conventions, Icon Convention, Feature File Order & Layout, New Feature Checklist, Execution Conventions, Supabase DB Query Convention
- `CLAUDE.md`: Do NOT Touch, Key Architecture Rules, Supabase Project

### Phase 4: Write verification plan

After implementation, re-run the 3 reference-extraction commands from the research report and confirm:
- Zero references to `plan/yawnly-cmp-implementation-plan.md` (except historical mentions in research report or this packet)
- Zero references to `plan/yawnly-prd-v3.md` or `plan/yawnly-prd-v4.md`
- Zero references to `plan/release-v1.1-findings.md`
- `docs/llm-agent-feature-implementation-guide.md` mentions remain only as acknowledged migration notices

## Common Failure Handling

- **Wiki paths break.** If MyWiki moves or is renamed, the bridge refs become stale themselves. Mitigation: use absolute wiki paths (`MyWiki/projects/yawnly/context.md`) so the reference is explicit.
- **AGENTS.md truncated by tool.** The file is ~154 lines. Read in chunks if needed.
- **CLAUDE.md read-order replacement is too aggressive.** The Read Order section is the agent boot sequence — if the replacement is unclear, agents may skip context. The replacement MUST be equally prescriptive and ordered.
- **agent-architecture.md is a canonical reference for non-Hermes agents (Claude, Cursor).** Edits to it must not break the doc's utility for those agents. Wiki refs should be additive, not replacement-only.
- **release-pending.md and komodo files are plan artifacts.** Treat them as working documents — edits should be minimal and preserve their utility as implementation specs.
- **If any edit would remove a Yawnly-specific convention that the wiki does not cover**, stop and flag it. The wiki intentionally does not duplicate build commands, UI token lists, or review gates.

## Allowed Paths

Hermes may **read**:
- `/Users/eloelo/Downloads/Yawnly/AGENTS.md`
- `/Users/eloelo/Downloads/Yawnly/CLAUDE.md`
- `/Users/eloelo/Downloads/Yawnly/docs/agent-architecture.md`
- `/Users/eloelo/Downloads/Yawnly/plan/v2/pending/release-pending.md`
- `/Users/eloelo/Downloads/Yawnly/plan/v2/komodo/komodo_integration_impl_spec.md`
- `/Users/eloelo/Downloads/Yawnly/modal/impl.md`
- `/Users/eloelo/Downloads/MyWiki/index.md`
- `/Users/eloelo/Downloads/MyWiki/projects/yawnly/context.md`
- `/Users/eloelo/Downloads/MyWiki/projects/yawnly/repo-map.md`
- `/Users/eloelo/Downloads/MyWiki/inbox/hermes-proposals/yawnly-stale-doc-reference-map-20260502.md`

Hermes may **write** only:
- `/Users/eloelo/Downloads/MyWiki/inbox/hermes-proposals/` (the plan output)

Hermes must **not** edit:
- Any file under `/Users/eloelo/Downloads/Yawnly/`
- Any canonical MyWiki page (anything outside `inbox/`)

## Required Context

- `inbox/hermes-proposals/yawnly-stale-doc-reference-map-20260502.md` (the research report)
- `projects/yawnly/context.md`
- `projects/yawnly/repo-map.md`
- `MyWiki/index.md` (First Read Protocol section, lines 13–21)
- `meta/task-packet-schema.md`

## Non-Goals

- Do not edit Yawnly source code.
- Do not edit Yawnly markdown files (this is plan-only).
- Do not edit canonical wiki pages.
- Do not modify `docs/agent-architecture.md` sections other than lines 5, 364, 365 (the stale refs).
- Do not touch any sections listed in Phase 3 (Yawnly-specific conventions the wiki does not duplicate).
- Do not propose architectural changes to Yawnly.
- Do not create new documentation files in the Yawnly repo.

## Stop Conditions

Stop and ask the human if any are true:
- The task wants edits outside `Allowed Paths`.
- A section that should be preserved (Phase 3 list) would be affected.
- A wiki path reference is needed but the target wiki page does not exist.
- The CLAUDE.md Read Order replacement cannot be made equally clear.
- Any edit touches more than 3 unexpected files beyond the 6 listed.
- The komodo or Modal references need a decision and the human hasn't provided one (flag them for review, don't assume).

## Output Required

- Path of the plan file (under `inbox/hermes-proposals/`).
- Count of files in scope and total lines that would change.
- Per-file edit table with before/after snippets.
- Human-input-needed table (komodo, Modal).
- Verification commands to run after implementation.
- List of preserved sections (evidence that Yawnly-specific knowledge survives).
- Risks / open questions.
- Suggested wiki updates (if any; these stay in `inbox/hermes-proposals/`).

## Dry Run Order (Yawnly Pilot)

1. ~~`research-only` on stale-doc packet~~ — done (`yawnly-stale-doc-reference-map-20260502.md`)
2. **`plan-only` on this packet** — Hermes drafts the bridge-thinning plan into `inbox/hermes-proposals/`
3. Human review of the plan.
4. If approved, promote to `implement-local` — Hermes edits the 6 files on a branch.
5. Re-run verification commands, confirm zero stale refs.
6. Human review of the diff before any `implement-pr`.
