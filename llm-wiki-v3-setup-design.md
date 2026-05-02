# Design Document — LLM Wiki v3 Setup + Yawnly Pilot

## Enforcement Contract

1. This design implements `llm-wiki-v3-setup-requirements.md`.
2. This design is documentation/setup planning only. It does not authorize source-code changes in Yawnly.
3. Risky claims are labeled `Verified` or `Out of Scope`.
4. Canonical source for the strategy is `llm-wiki-plan-v3.md`.

---

## Overview

The implementation will create a minimal Markdown wiki at `/Users/eloelo/Downloads/MyWiki` and use Yawnly as the first pilot project.

The system has three layers:

1. **Stable wiki layer** — global guidelines, project context, project decisions, golden examples, research conclusions, task packets.
2. **Hermes execution layer** — Telegram control, memory, skills, sessions, cron, branch execution, reports.
3. **Project repo layer** — source code, build files, project-local docs, tests, runtime behavior.

The wiki will not duplicate Hermes memory, skills, cron, or logs. The wiki defines what Hermes should trust and what it is allowed to do.

---

## Steering Alignment

### Spec Workflow Alignment

Verified 2026-05-02 from `/Users/eloelo/Downloads/Yawnly/.spec-workflow/steering/tech.md`:

- Spec truth must live in files.
- No implementation starts until red-team pass reports zero `P0/P1`.
- Tasks must define exact files, commands, pass/fail output, and rollback notes for runtime-risk work.

This design follows that workflow by producing three planning files:

- `llm-wiki-v3-setup-requirements.md`
- `llm-wiki-v3-setup-design.md`
- `llm-wiki-v3-setup-tasks.md`

### LLM Wiki v3 Alignment

Verified 2026-05-02 from `/Users/eloelo/Downloads/MyWiki/llm-wiki-plan-v3.md`:

- Hermes is the active operating system.
- The wiki is the stable constitution.
- Yawnly is the first pilot.
- The task packet is the contract between the wiki and Hermes.

---

## Target File System Design

### Launch Root

```text
/Users/eloelo/Downloads/MyWiki/
```

If the human chooses a different root before execution, `llm-wiki-v3-setup-tasks.md` must be patched before implementation begins.

### Launch Tree

```text
MyWiki/
├── index.md
├── README.md
├── AGENTS.md
├── HERMES.md
├── guidelines/
│   ├── frontend/
│   ├── backend/
│   ├── shared/
│   └── agents/
├── golden-examples/
│   ├── cmp/
│   ├── supabase/
│   ├── fastapi/
│   └── spring/
├── projects/
│   ├── _template/
│   └── yawnly/
│       └── task-packets/
├── research/
│   ├── inbox/
│   ├── accepted/
│   └── rejected/
├── inbox/
│   ├── quick-capture/
│   ├── hermes-proposals/
│   └── claude-proposals/
└── meta/
```

No other top-level folders are created in Phase 1.

---

## Component Design

### Component 1 — Root Agent Entry Files

#### `index.md`

Purpose: first-read map for all agents.

Required structure:

```md
# LLM Wiki Index

## First Read Protocol
1. Read this file.
2. Read the active project's `context.md`.
3. Read the active project's `linked-guidelines.md`.
4. Read only the linked guideline and golden-example pages needed for the task.
5. Read the current task packet.

## Active Projects
- `projects/yawnly/context.md` — CMP bedtime story app; first pilot.

## Global Guidelines
- `guidelines/frontend/compose-multiplatform.md`
- `guidelines/frontend/compose-ui.md`
- `guidelines/backend/supabase.md`
- `guidelines/backend/supabase-edge-functions.md`
- `guidelines/agents/hermes-delegation.md`

## Do Not Bulk Load
See `meta/wikiignore.md`.
```

Pass condition:

- `index.md` stays under 150 lines.
- It links to Yawnly context and linked guidelines.

#### `README.md`

Purpose: human-facing explanation.

Required sections:

- What this wiki is
- What this wiki is not
- How to use it with Hermes
- How to add a project
- How to promote research

#### `AGENTS.md`

Purpose: shared base instructions for agents.

Required statements:

- Read `index.md` first.
- Do not recursively load the wiki.
- Use task packets for non-trivial work.
- Write proposals to `inbox/*-proposals/`.
- Do not write canonical docs without human approval.

#### `HERMES.md`

Purpose: Hermes-specific instructions.

Required statements:

- Telegram commands default to `plan-only`.
- Implementation requires approved task packet.
- Branch naming: `hermes/<project>-<task>`.
- Stop when task exceeds allowed paths.
- Use `inbox/hermes-proposals/` for wiki updates.

Out of Scope:

- `CLAUDE.md` at wiki root. Claude/Cursor can follow `AGENTS.md` during Phase 1. Add `CLAUDE.md` later only if tool behavior requires it.

---

### Component 2 — Meta Schemas

#### `meta/page-schema.md`

Purpose: canonical page metadata.

Canonical schema:

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

Project-file minimal schema:

```yaml
---
title:
summary:
type:
status:
updated:
---
```

Invariant:

- Page frontmatter is not modified only to record reads.

#### `meta/task-packet-schema.md`

Purpose: exact Hermes work-order contract.

Required sections:

- Goal
- Why
- End Goal
- Output Format
- Recommended Approach
- Common Failure Handling
- Allowed Paths
- Required Context
- Reference Files In Repo
- Non-Goals
- Implementation Plan
- Verification
- Stop Conditions
- Output Required

Default mode:

- `plan-only`

#### `meta/research-note-schema.md`

Purpose: research distillation format.

Required sections:

- Question
- Keep
- Decision
- Rejected
- Applies To
- Next Action
- Sources

#### `meta/wikiignore.md`

Purpose: tell agents what not to bulk-load.

Required entries:

```md
# Wiki Ignore

Agents must not bulk-load:
- `research/rejected/`
- `inbox/`
- `golden-examples/`
- generated reports copied into project folders
- archived/stale pages
- task packets not linked by the current project context

Agents may read ignored paths only when a task packet explicitly lists the path.
```

---

### Component 3 — Global Guidelines

#### `guidelines/frontend/compose-multiplatform.md`

Sources:

- Verified 2026-05-02: `Yawnly/AGENTS.md`
- Verified 2026-05-02: `Yawnly/docs/agent-architecture.md`
- Verified 2026-05-02: `Yawnly/CLAUDE.md`

Required content:

- CMP mental model
- module/source-set expectations
- feature folder structure
- clean architecture dependency direction
- ViewModel → UseCase → Repository → RepositoryImpl → ApiService
- DTO/domain separation
- Koin module placement and platform registration
- core-to-feature dependency rule
- logging rule
- build verification examples

Yawnly-specific details must be excluded unless labeled as an example.

#### `guidelines/frontend/compose-ui.md`

Sources:

- Verified 2026-05-02: `Yawnly/DESIGN.md`
- Verified 2026-05-02: `Yawnly/docs/yawnly-ui-ux-polish-guide.md`
- Verified 2026-05-02: `Yawnly/AGENTS.md`

Required content:

- Root/Content split
- state/action/side-effect pattern
- side-effect collection rule
- token-first dimensions/colors
- tap handler convention
- top-level screen composition checklist
- animation caution: purposeful, not broad automatic motion

Yawnly brand colors and bedtime styling stay in `projects/yawnly/context.md` or `projects/yawnly/decisions.md`.

#### `guidelines/backend/supabase.md`

Sources:

- Verified 2026-05-02: `Yawnly/AGENTS.md`
- Verified 2026-05-02: `Yawnly/docs/agent-architecture.md`
- Verified 2026-05-02: `Yawnly/plan/v2/komodo/komodo_supabase_wiring_map.md`

Required content:

- client anon key vs service role boundary
- mobile client calls vs Edge Functions
- RLS expectations
- schema verification before SQL
- no raw secrets
- DTO/function/migration alignment
- subscriptions/billing truth belongs server-side for billing flows

#### `guidelines/backend/supabase-edge-functions.md`

Sources:

- Verified 2026-05-02: `Yawnly/plan/otp_integration.md`
- Verified 2026-05-02: `Yawnly/plan/v2/komodo/supabase_iap_server_impl_spec.md`
- Verified 2026-05-02: `Yawnly/plan/v2/komodo/komodo_supabase_wiring_map.md`

Required content:

- function file structure
- env secret handling
- request/response envelope conventions
- JWT/auth requirements
- error normalization
- webhook/idempotency caution
- migration alignment

#### `guidelines/agents/hermes-delegation.md`

Sources:

- Verified 2026-05-02: `llm-wiki-plan-v3.md`
- Verified 2026-05-02: Hermes Atlas guide

Required content:

- Hermes owns execution, memory, skills, Telegram, cron, logs
- wiki owns stable truth and task boundaries
- delegation modes
- task packet requirement
- DeepSeek-first provider rule for research/planning
- OpenAI-later rule for risky implementation/review
- Curator checkpoint for Hermes skill cleanup
- allowed paths rule
- branch rule
- proposal lane rule
- model selection guidance for high-risk tasks

---

### Component 4 — Yawnly Project Folder

#### `projects/yawnly/context.md`

Required content:

- Product: CMP bedtime story app for Indian kids.
- Repo path: `/Users/eloelo/Downloads/Yawnly`.
- Main shared module: `composeApp`.
- Android app entry: `androidApp`.
- iOS host: `iosApp`.
- Backend: `supabase/`.
- Important feature packages: onboarding, tonightsstory, storygenerator, storyreader, mythology, paywall, profile, sharecard.
- Hard constraints:
  - Supabase is product backend.
  - Ktor/BaseRepository is legacy skeleton path for non-Supabase flows.
  - no Room for new Yawnly product data.
  - no changes to `BaseViewModel`, `BaseRepository`, `BaseResponseDto`, or theme/core infrastructure unless task explicitly permits it.
- Build commands:
  - `rtk ./gradlew :composeApp:compileKotlinAndroid`
  - `rtk ./gradlew :androidApp:assembleDebug`

#### `projects/yawnly/linked-guidelines.md`

Required links:

- `guidelines/frontend/compose-multiplatform.md`
- `guidelines/frontend/compose-ui.md`
- `guidelines/backend/supabase.md`
- `guidelines/backend/supabase-edge-functions.md`
- `guidelines/agents/hermes-delegation.md`

#### `projects/yawnly/decisions.md`

Seed decisions:

1. Supabase is Yawnly's product backend.
2. Ktor/BaseRepository is legacy skeleton and not the path for new Yawnly product APIs.
3. No Room for new Yawnly product data.
4. Ship audio/story flow before 3D/Lottie buddy.
5. Structured prompt/entropy approach, not RAG, for story generation.
6. PII placeholders in model-facing prompts.
7. Subscription truth should be server-authoritative.
8. Story Library/retention work has higher near-term product value than heavy visual buddy work.

Each decision uses:

- ID
- Status
- Date
- Decision
- Why
- Rejected
- Revisit

#### `projects/yawnly/current-state.md`

Required sections:

- Working
- In Progress
- Blocked
- Risks
- Next Candidate Task Packets

#### `projects/yawnly/repo-map.md`

Required sections:

- Actual modules
- Feature packages
- Core packages
- Supabase functions/migrations
- Important docs
- Missing/stale references
- Generated reports to avoid by default

Verified stale/reference issues to list:

- `plan/yawnly-cmp-implementation-plan.md` is referenced by docs but was not found during this review.
- PRD references such as `plan/yawnly-prd-v3.md` or `plan/yawnly-prd-v4.md` were referenced but not found during this review.
- `iosApp` is present as Xcode host, not a Gradle module.

#### `projects/yawnly/backlog.md`

Allowed sections:

- release-critical
- next
- later
- parked

No more than 150 lines.

#### `projects/yawnly/task-packets/README.md`

Required content:

- task packet usage
- status meanings
- approved modes
- review rules

#### First task packet

Recommended first packet:

```text
projects/yawnly/task-packets/yawnly-20260502-stale-doc-reference-map.md
```

Mode:

- `plan-only`

Goal:

- produce a no-code report/task packet for stale Yawnly references and agent bridge cleanup.

Allowed paths:

- `projects/yawnly/`
- future Yawnly repo bridge files only after human approval

This first packet should not edit Yawnly source code.

---

### Component 5 — Golden Examples

Phase 1 creates empty folders only:

```text
golden-examples/cmp/
golden-examples/supabase/
golden-examples/fastapi/
golden-examples/spring/
```

No example files are seeded until a real source file or pattern is selected.

Reason:

- empty folders define future homes without inventing examples.
- fake examples would become worse than no examples.

---

## Hermes Setup Design

Hermes setup is planned after the wiki skeleton and Yawnly context exist.

Detailed installation/provider plan:

- `/Users/eloelo/Downloads/MyWiki/hermes-installation-plan.md`

Required Hermes constraints:

- access to `/Users/eloelo/Downloads/MyWiki`
- access to `/Users/eloelo/Downloads/Yawnly`
- access to a scratch directory
- no day-job repo access
- branch-only edits
- destructive-operation approvals enabled
- Telegram allowlist using numeric Telegram user ID
- DeepSeek v4 Pro tried first when available
- OpenAI added later through a separate provider/fallback task packet
- Hermes Curator status checked before implementation work
- mission-critical custom skills pinned before manual Curator runs

Out of Scope:

- Installing Hermes in this spec pass.
- Running Hermes before the wiki skeleton exists.
- Letting Hermes edit multiple projects before Yawnly succeeds.
- Building a MyWiki replacement for Hermes Curator, Hermes memory, or Hermes skills.

---

## Data Models

### Page Metadata

Defined in `meta/page-schema.md`.

### Task Packet Metadata

Defined in `meta/task-packet-schema.md`.

### Research Metadata

Defined in `meta/research-note-schema.md`.

No runtime application data models are created.

---

## Error Handling

### Error Scenario 1 — Wiki Root Already Exists

Trigger:

- `/Users/eloelo/Downloads/MyWiki` exists before Phase 1.

Expected behavior:

- Stop.
- Read existing tree.
- Produce a collision report.
- Ask human whether to merge, rename, or delete.

Recovery:

- No destructive deletion without explicit approval.

### Error Scenario 2 — Yawnly Source Docs Contradict Each Other

Trigger:

- Two Yawnly docs disagree about a product or architecture rule.

Expected behavior:

- Prefer actual code/repo-map facts for structure.
- Prefer newer implemented specs for deployed behavior.
- Mark unresolved contradiction in `projects/yawnly/current-state.md` or a task packet.

Recovery:

- Do not encode the contradiction as a canonical global guideline.

### Error Scenario 3 — Extracted Rule Is Project-Specific

Trigger:

- A rule includes Yawnly product/brand/scope specifics.

Expected behavior:

- Move it to `projects/yawnly/context.md` or `projects/yawnly/decisions.md`.

Recovery:

- Keep global guideline reusable.

### Error Scenario 4 — Agent Proposal Becomes Stale

Trigger:

- Proposal is older than 14 days.

Expected behavior:

- Promote, delete, or archive during weekly maintenance.

Recovery:

- Do not leave stale proposals as hidden context.

---

## Testing Strategy

### Documentation Verification

Commands for future implementation:

```shell
rg "plan/yawnly-cmp-implementation-plan|yawnly-prd" "/Users/eloelo/Downloads/Yawnly" --glob "*.md"
rg "SupabaseBaseRepository|BaseRepository|@Serializable|Room|Ktor" "/Users/eloelo/Downloads/Yawnly" --glob "*.md"
rg "3D buddy|audio-only|Story Library|PII|placeholder|RAG" "/Users/eloelo/Downloads/Yawnly" --glob "*.md"
```

Pass condition:

- Findings from these commands are reflected in Yawnly project context, decisions, or repo-map.

### Structure Verification

Commands for future implementation:

```shell
test -f "/Users/eloelo/Downloads/MyWiki/index.md"
test -f "/Users/eloelo/Downloads/MyWiki/projects/yawnly/context.md"
test -f "/Users/eloelo/Downloads/MyWiki/projects/yawnly/decisions.md"
test -f "/Users/eloelo/Downloads/MyWiki/meta/task-packet-schema.md"
```

Pass condition:

- All commands exit with code `0`.

### Scope Verification

Commands for future implementation:

```shell
test ! -d "/Users/eloelo/Downloads/MyWiki/ops"
test ! -f "/Users/eloelo/Downloads/MyWiki/TASKS.md"
test ! -d "/Users/eloelo/Downloads/MyWiki/prompts"
```

Pass condition:

- All commands exit with code `0`.

### Agent Read Verification

Manual check:

1. Read `index.md`.
2. Follow only the Yawnly links.
3. Confirm the read path reaches task packet instructions without bulk-loading unrelated folders.

Pass condition:

- The read path is deterministic and under 6 files before task-specific context.

---

## Rollout

Rollout has five gates:

1. Spec approval.
2. Wiki skeleton.
3. Yawnly context and decisions.
4. Hermes setup for Yawnly only.
5. One complete task-packet loop.

No other CMP project enters the wiki until gate 5 passes.