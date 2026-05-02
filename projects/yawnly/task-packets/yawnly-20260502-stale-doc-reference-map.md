---
id: yawnly-20260502-stale-doc-reference-map
project: yawnly
repo_path: /Users/eloelo/Downloads/Yawnly
status: draft
mode: plan-only
risk: low
created: 2026-05-02
owner: human
executor: hermes
---

# Task: Stale Doc Reference Map (Yawnly)

## Goal

Produce a no-code report that maps every stale or missing documentation reference inside the Yawnly repo, then propose either (a) a follow-up `plan-only` packet to thin the Yawnly bridge files (`AGENTS.md`, `CLAUDE.md`), or (b) a follow-up `plan-only` packet to re-create the missing primary references — depending on what the report finds.

## Why

Yawnly's agent docs cite primary references that no longer exist. If Hermes (or any coding agent) trusts these stale references, it works from false context. The pilot must start from clean documentation.

Verified examples (2026-05-02):
- `Yawnly/CLAUDE.md` (lines 17 and 23) and `Yawnly/AGENTS.md` (line 126) point at `plan/yawnly-cmp-implementation-plan.md`, which does not exist in the repo.
- `Yawnly/docs/agent-architecture.md` notes that `docs/llm-agent-feature-implementation-guide.md` was removed; bookmarks elsewhere may still cite the removed path.

## End Goal

A markdown report stored in `inbox/hermes-proposals/` that:
1. Lists every Yawnly markdown reference that points at a missing or known-stale file.
2. For each finding, records the citing file and line, the cited target, and the actual filesystem state (`exists`, `missing`, `superseded-by:<path>`).
3. Recommends one follow-up action per finding: keep, delete the citation, replace with the wiki path, re-create the missing doc, or hold for human input.
4. Closes with a draft of either:
   - `task-packets/yawnly-YYYYMMDD-bridge-thinning.md` — thins `AGENTS.md`/`CLAUDE.md` to point at MyWiki, or
   - `task-packets/yawnly-YYYYMMDD-recreate-implementation-plan.md` — proposes a structure for re-creating `plan/yawnly-cmp-implementation-plan.md`.

## Output Format

A single markdown file under `inbox/hermes-proposals/` named `yawnly-stale-doc-reference-map-YYYYMMDD.md`. No source code edits. No canonical wiki edits.

## Recommended Approach

1. Read `projects/yawnly/context.md` and `projects/yawnly/repo-map.md`. Confirm the assumptions still hold.
2. Run reference-extraction over Yawnly markdown:
   - `rg -n "\[.*\]\(([^)]+\.md)\)" /Users/eloelo/Downloads/Yawnly --glob "*.md"` for markdown links.
   - `rg -n "\`(plan/[^\`]+\.md|docs/[^\`]+\.md)\`" /Users/eloelo/Downloads/Yawnly --glob "*.md"` for backticked filename mentions.
   - `rg -n "yawnly-cmp-implementation-plan|yawnly-prd|llm-agent-feature-implementation-guide" /Users/eloelo/Downloads/Yawnly --glob "*.md"` for known-suspect names.
3. For each unique cited path, run `test -f` to record `exists` vs `missing`.
4. For docs that exist but are flagged as superseded (e.g. `docs/codebase-review.md` is dated; `docs/llm-agent-feature-implementation-guide.md` is removed), record the superseder.
5. Group findings by citing file. Propose a single follow-up packet per high-volume citer.
6. Hand the report to the human for review.

## Common Failure Handling

- If `rg` returns thousands of matches because of generated reports under `graphify-out/`, exclude those paths with `--glob '!**/graphify-out/**' --glob '!**/build/**' --glob '!**/.gradle/**'`.
- If a referenced file looks present but is actually a directory or symlink, record the actual filesystem type explicitly.
- If a citation uses a relative path that resolves outside the repo, do not follow it; flag and skip.
- If the report grows past 200 findings, stop and ask the human whether to scope by directory rather than continuing.

## Allowed Paths

Hermes may **read**:
- `/Users/eloelo/Downloads/Yawnly/**/*.md`
- `/Users/eloelo/Downloads/MyWiki/projects/yawnly/**`
- `/Users/eloelo/Downloads/MyWiki/index.md`
- `/Users/eloelo/Downloads/MyWiki/AGENTS.md`
- `/Users/eloelo/Downloads/MyWiki/HERMES.md`
- `/Users/eloelo/Downloads/MyWiki/meta/**`
- `/Users/eloelo/Downloads/MyWiki/guidelines/agents/hermes-delegation.md`

Hermes may **write** only:
- `/Users/eloelo/Downloads/MyWiki/inbox/hermes-proposals/`
- `/Users/eloelo/Downloads/hermes-scratch/` (for intermediate notes)

Hermes must **not** edit:
- any file under `/Users/eloelo/Downloads/Yawnly/`
- any canonical MyWiki page (anything outside `inbox/`)

## Required Context

- `projects/yawnly/context.md`
- `projects/yawnly/linked-guidelines.md`
- `projects/yawnly/repo-map.md`
- `meta/task-packet-schema.md`
- `guidelines/agents/hermes-delegation.md`

## Non-Goals

- Do not edit Yawnly source code.
- Do not edit Yawnly markdown.
- Do not promote canonical wiki pages.
- Do not propose architectural changes to Yawnly.
- Do not include the contents of `graphify-out/` or `build/` artifacts in the report.

## Stop Conditions

Stop and ask the human if any are true:
- The task seems to need edits outside `Allowed Paths` (including any Yawnly file).
- A reference resolves to a file that contains secrets — escalate immediately, do not paste secrets into the report.
- More than 200 distinct stale references are found (scope the work first).
- A reference depends on a file outside the Yawnly repo.
- More than 3 unexpected files need edits.
- Architecture differs from the linked guidelines (in this packet, "architecture" includes the proposal-only / no-code-edits invariant).

## Output Required

- Path of the report file (under `inbox/hermes-proposals/`).
- Count of citing files and total stale references.
- The list of grouped findings (citer, target, actual state, recommendation).
- A draft follow-up packet (one of bridge-thinning OR recreate-implementation-plan, whichever the findings imply).
- Risks / open questions.
- Suggested wiki updates, if any (these stay in `inbox/hermes-proposals/`).

## Dry Run Order (Yawnly Pilot)

1. `research-only` on this packet — Hermes returns a report only.
2. Human review.
3. If clear, `plan-only` — Hermes drafts the follow-up packet into `inbox/hermes-proposals/`.
4. Human review and approval before any future `implement-local` work.
