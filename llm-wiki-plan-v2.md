# Personal LLM Wiki v2 — Hermes Control System Plan

**Status:** Draft for review  
**Date:** 2026-05-02  
**Purpose:** Build a simple, durable LLM-readable engineering wiki that helps Hermes, Claude Code, Cursor, and other agents work across multiple personal projects with less repeated explanation and less review of agent slop.

---

## 1. Core conclusion

The v1 plan was directionally good, but it was too close to rebuilding a second Hermes around Hermes.

The wiki should not become:

- a custom agent memory engine
- a custom task manager
- a quota platform
- a huge second brain
- an auto-generated documentation system
- a replacement for Hermes skills, cron, gateways, MCP, or internal memory

The wiki should be much smaller:

> A file-backed engineering control layer that tells agents what standards to follow, what each project is, what decisions were made, and exactly what task they are allowed to execute.

Hermes is the executor. Telegram is the control center. Git branches are the review surface. The wiki is the source of truth and context router.

---

## 2. The real problem to solve

The problem is not lack of documentation. Yawnly already has many docs:

- `AGENTS.md`
- `CLAUDE.md`
- `DESIGN.md`
- `docs/agent-architecture.md`
- `docs/yawnly-ui-ux-polish-guide.md`
- `.spec-workflow/steering/tech.md`
- `plan/v2/pending/*`
- `plan/otp_integration.md`
- generated graph reports
- agent memory files

The problem is that knowledge is split and duplicated:

- reusable CMP rules live inside one project
- project-specific decisions are mixed with general architecture rules
- product decisions are scattered across plan files
- some files reference docs that are missing or stale
- agents read too much or the wrong things
- every project requires repeated explanation
- reviewing broad agent-generated diffs wastes time

This wiki solves that by separating:

| Knowledge type | Where it belongs |
|---|---|
| Reusable FE/BE standards | `guidelines/` |
| Concrete reusable examples | `golden-examples/` |
| Research conclusions worth keeping | `research/accepted/` |
| Project-specific context | `projects/<project>/context.md` |
| Project-specific decisions | `projects/<project>/decisions.md` |
| Current project state | `projects/<project>/current-state.md` |
| Work delegated to Hermes | `projects/<project>/task-packets/` |
| Temporary notes and agent proposals | `inbox/` |

---

## 3. Workload this system is designed for

Current and near-future projects:

- 4 Compose Multiplatform apps
- 1 Spring backend
- Yawnly with Supabase backend and Edge Functions
- 1 new CMP project using Python FastAPI + Supabase backend
- future web frontends, DevOps, self-hosted services, and vendor/quota tracking

This means the wiki must support:

- reusable CMP architecture rules
- reusable Supabase patterns
- reusable FastAPI backend patterns
- reusable Spring backend patterns
- project-level product and architecture decisions
- research distillation
- agent task delegation from Telegram
- strict per-project execution boundaries

The wiki must stay small enough that it can be maintained by one developer with a full-time job.

---

## 4. Design principles

### 4.0 Community patterns this plan keeps

This v2 plan should not ignore the useful community work from v1. It keeps the strongest patterns from Karpathy's LLM-Wiki idea, the DAIR.AI breakdown, practical Git/Markdown wikis, and agent context-file conventions.

References that should continue to inform the plan:

- [Karpathy LLM-Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — core inspiration: central Markdown wiki, `index.md`, `log.md`, agent-readable memory, optional MCP/search.
- [DAIR.AI breakdown of Karpathy pattern](https://academy.dair.ai/blog/llm-knowledge-bases-karpathy) — useful explanation of ingest, compile, query, and lint loops; reinforces that simple Markdown works before RAG.
- [AGENTS.md](https://agents.md/) — cross-agent context-file convention; useful as the thin bridge into the wiki.
- [OpenAI Codex AGENTS.md docs](https://developers.openai.com/codex/guides/agents-md/) — useful for nested context and agent instruction precedence.
- [qmd](https://github.com/tobi/qmd) — likely later search/MCP layer if `rg` stops being enough.
- [llms.txt](https://llmstxt.org/) — useful style pattern for concise LLM-readable indexes.
- [SamurAIGPT/llm-wiki-agent](https://github.com/SamurAIGPT/llm-wiki-agent) — practical implementation reference.
- [Ar9av/obsidian-wiki](https://github.com/Ar9av/obsidian-wiki) — practical Obsidian/wiki implementation reference.

Patterns kept:

- **Markdown-first, git-backed knowledge** — plain files remain the canonical source.
- **`index.md` as the first read** — agents start from a small map, not a recursive repo load.
- **Raw notes are not equal to canonical knowledge** — capture freely, then distill/promote.
- **Search starts simple** — `rg`, filenames, frontmatter, and a good index before vector/RAG tools.
- **Agents can help maintain the wiki** — but by proposing updates, not silently rewriting truth.
- **Small pages beat large docs** — each page should answer one recurring question.
- **Link out, do not copy external docs** — the wiki stores your opinionated take, deviations, and gotchas.
- **MCP/search tools are later accelerators** — useful after the corpus is disciplined, not before.

What v2 changes from the community examples:

- Karpathy-style LLM-Wiki can be very broad; this version is narrower because the main goal is engineering execution across your projects.
- Agent-written wiki updates are gated by human promotion because bad canonical guidance can make Hermes worse.
- Project decisions stay project-local instead of becoming one large global knowledge base.
- Hermes task packets become the execution layer, because your pain is not only retrieval; it is controlling what agents are allowed to do.

Review rule:

> Community patterns should inform the system, but anything added must reduce repeated explanation, reduce review burden, or improve Hermes task safety. If it does not help one of those, it stays out.

### 4.1 Plain Markdown is the source of truth

The canonical store is a git repo with Markdown files and frontmatter. Other tools may read or index it, but no tool owns it.

Allowed layers:

- Obsidian/Foam/Cursor as editors
- `rg` for search
- markdown lint/link checks later
- `qmd` later if search becomes painful
- Hermes/Claude/Cursor as readers and proposal writers

Not allowed as canonical backends:

- Notion
- BookStack
- Outline
- Wiki.js
- custom vector DB
- database-first personal knowledge system

### 4.2 Global guidelines are separate from projects

FE/BE standards should live outside every repo. Projects should reference them and only document deviations.

Example:

```md
Use:
- `guidelines/frontend/compose-multiplatform.md`
- `guidelines/backend/supabase.md`
- `guidelines/shared/api-contracts.md`

Yawnly deviations:
- Product APIs use Supabase; legacy Ktor skeleton code is retained but not used for new product work.
- Product data is online-only; Room is not used for Yawnly product data.
```

### 4.3 Most decisions are project-local

Not every decision is global. A decision belongs where its blast radius is.

Project decision:

- affects one app
- lives in `projects/<project>/decisions.md`
- example: Yawnly ships audio-only before 3D buddy

Global decision:

- affects multiple projects
- lives in `decisions/global/`
- example: all CMP apps use Root/Content screen split and DTO/domain separation

### 4.4 Research is distilled, not dumped

Research notes should preserve conclusions, not the whole internet.

The wiki should answer:

- What did we learn?
- What did we decide?
- What did we reject?
- What should agents do next time?

Raw links can be included, but accepted research should be short.

### 4.5 Agents propose, humans promote

Hermes and other agents may write:

- `inbox/`
- `logs/`
- task reports
- branch changes
- proposed updates

Agents must not directly overwrite canonical guidelines or project decisions unless explicitly asked.

### 4.6 Task packets are the delegation unit

Hermes should not be told:

> Go improve Yawnly.

Hermes should be told:

> Execute this task packet in this project, within these paths, following these guidelines, on this branch, and stop after this verification.

This is how review stays manageable.

---

## 5. Proposed v2 folder structure

```text
llm-wiki/
├── index.md
├── README.md
├── AGENTS.md
├── CLAUDE.md
├── HERMES.md
│
├── guidelines/
│   ├── frontend/
│   │   ├── compose-multiplatform.md
│   │   ├── compose-ui.md
│   │   └── web-frontend.md
│   ├── backend/
│   │   ├── supabase.md
│   │   ├── supabase-edge-functions.md
│   │   ├── fastapi.md
│   │   └── spring-kotlin.md
│   ├── shared/
│   │   ├── api-contracts.md
│   │   ├── auth.md
│   │   ├── error-handling.md
│   │   ├── testing.md
│   │   └── security.md
│   └── agents/
│       ├── delegation-protocol.md
│       ├── telegram-control.md
│       └── review-report-format.md
│
├── golden-examples/
│   ├── cmp/
│   │   ├── screen-root-content-pattern.md
│   │   ├── clean-feature-module.md
│   │   ├── koin-feature-module.md
│   │   └── supabase-repository-pattern.md
│   ├── supabase/
│   │   ├── edge-function-pattern.md
│   │   ├── migration-rls-pattern.md
│   │   └── webhook-handler-pattern.md
│   ├── fastapi/
│   │   ├── project-layout.md
│   │   ├── auth-middleware.md
│   │   └── service-repository-pattern.md
│   └── spring/
│       ├── controller-service-repository.md
│       └── validation-error-pattern.md
│
├── research/
│   ├── inbox/
│   ├── accepted/
│   └── rejected/
│
├── decisions/
│   └── global/
│
├── projects/
│   ├── yawnly/
│   │   ├── context.md
│   │   ├── decisions.md
│   │   ├── current-state.md
│   │   ├── backlog.md
│   │   ├── repo-map.md
│   │   ├── linked-guidelines.md
│   │   └── task-packets/
│   ├── _template/
│   │   ├── context.md
│   │   ├── decisions.md
│   │   ├── current-state.md
│   │   ├── backlog.md
│   │   ├── repo-map.md
│   │   ├── linked-guidelines.md
│   │   └── task-packets/
│   └── README.md
│
├── inbox/
│   ├── research/
│   ├── agent-proposals/
│   └── quick-capture/
│
├── logs/
│   └── YYYY-MM.md
│
├── meta/
│   ├── page-schema.md
│   ├── task-packet-schema.md
│   ├── research-note-schema.md
│   ├── wikiignore.md
│   ├── references.log
│   └── maintenance-checklist.md
│
└── ops/
    ├── vendors/
    ├── quotas/
    └── servers/
```

Launch should not create every file above. This structure defines where things go as the wiki grows.

---

## 6. First-read behavior for agents

Agents should not recursively load the wiki.

Default read order:

1. `index.md`
2. relevant `projects/<project>/context.md`
3. `projects/<project>/linked-guidelines.md`
4. only the linked guideline files needed for the task
5. only the relevant golden example
6. current task packet

Never load:

- entire `projects/`
- entire `research/`
- entire `logs/`
- archived or rejected notes
- generated graph reports unless explicitly requested

---

## 7. Project directory contract

Each project gets a small wiki folder. This is not a duplicate of the repo. It is the agent-facing project control file set.

### 7.1 `context.md`

Purpose: tell agents what the project is and what matters.

Contents:

- product summary
- user/persona summary
- stack
- repo path
- build/test commands
- current production/staging status
- important constraints
- what not to touch
- project-specific deviations from global guidelines

Example Yawnly content:

- CMP bedtime story app for Indian kids
- shared code in `composeApp/src/commonMain`
- Android app entry in `androidApp`
- iOS host in `iosApp`
- backend in `supabase/`
- Supabase is live product backend
- legacy Ktor/BaseRepository skeleton exists but should not be used for new Yawnly product APIs
- product data is online-only; do not introduce Room for new Yawnly product features
- run `rtk ./gradlew :composeApp:compileKotlinAndroid` for shared compile
- run `rtk ./gradlew :androidApp:assembleDebug` for Android APK

### 7.2 `linked-guidelines.md`

Purpose: map the project to global rules.

Example:

```md
# Yawnly Linked Guidelines

Read these for CMP work:
- `guidelines/frontend/compose-multiplatform.md`
- `guidelines/frontend/compose-ui.md`
- `guidelines/backend/supabase.md`
- `guidelines/backend/supabase-edge-functions.md`
- `guidelines/shared/api-contracts.md`
- `guidelines/shared/error-handling.md`
- `guidelines/agents/delegation-protocol.md`

Use these golden examples:
- `golden-examples/cmp/screen-root-content-pattern.md`
- `golden-examples/cmp/supabase-repository-pattern.md`
- `golden-examples/supabase/edge-function-pattern.md`
```

### 7.3 `decisions.md`

Purpose: stop forgetting product and architecture decisions.

Use for:

- product scope choices
- launch sequencing
- pricing decisions
- stack choices specific to this project
- why something was deferred
- why a tempting approach was rejected

Format:

```md
## D001: Ship Audio-Only Before 3D Buddy

Status: accepted
Date: 2026-05-02
Scope: Yawnly

Decision:
Ship audio/story flow first. Do not build 3D/Lottie buddy before v1.

Why:
The buddy can delay launch and requires animation/rendering complexity. Validate payment, retention, and story quality first.

Rejected:
- Full 3D animation pipeline pre-launch
- Lottie buddy as launch blocker

Revisit:
After retention and payment are validated, or around 500 paying users.
```

### 7.4 `current-state.md`

Purpose: give agents a fresh snapshot without reading stale plans.

Contents:

- currently live features
- in-progress features
- broken/blocked areas
- current risks
- recent important changes
- next recommended tasks

This should be short and updated manually or via approved Hermes proposal.

### 7.5 `repo-map.md`

Purpose: quick orientation for the repo.

Contents:

- module map
- key directories
- backend locations
- test locations
- generated files to avoid
- known stale docs or dead references

For Yawnly, this should call out:

- Gradle modules: `composeApp`, `androidApp`, `komodo`
- `iosApp` is an Xcode host, not a Gradle module
- feature packages under `composeApp/src/commonMain/kotlin/com/techmo/yawnly/feature`
- core infrastructure under `core/feature`
- Supabase functions and migrations under `supabase/`
- missing/stale references from existing docs should not be trusted blindly

### 7.6 `backlog.md`

Purpose: a simple project-local backlog, not a replacement for GitHub issues.

Keep it short:

- release-critical
- next
- later
- parked

If it grows beyond 100-150 lines, move to GitHub/Linear or split into task packets.

### 7.7 `task-packets/`

Purpose: exact work orders for Hermes.

This is the most important part for reducing your involvement.

---

## 8. Task packet schema

Every Hermes task should start as a task packet.

```md
---
id: yawnly-YYYYMMDD-short-name
project: yawnly
repo_path: /path/to/Yawnly
status: draft | approved | running | review | done | cancelled
risk: low | medium | high
created: YYYY-MM-DD
owner: human
executor: hermes
---

# Task: Short Name

## Goal
What should change, in one clear paragraph.

## Why
The product/technical reason this matters.

## Allowed Paths
- `composeApp/src/commonMain/...`
- `supabase/functions/...`

Hermes must not edit paths outside this list.

## Required Context
Read:
- `projects/yawnly/context.md`
- `projects/yawnly/linked-guidelines.md`
- `guidelines/frontend/compose-multiplatform.md`
- `golden-examples/cmp/screen-root-content-pattern.md`

## Reference Files In Repo
- `path/to/good/example/File.kt`
- `path/to/current/feature/File.kt`

## Non-Goals
- Do not refactor unrelated files.
- Do not change public contracts unless listed.
- Do not update theme/core/base classes unless explicitly allowed.

## Implementation Plan
1. Step one.
2. Step two.
3. Step three.

## Verification
Commands:
- `rtk ./gradlew :composeApp:compileKotlinAndroid`
- `rtk ./gradlew :androidApp:assembleDebug`

Manual checks:
- Check this screen/flow.
- Confirm no regression in this path.

## Stop Conditions
Stop and ask human if:
- build requires dependency changes
- task touches paths outside allowed list
- secrets or credentials are needed
- architecture differs from required context
- more than 3 files outside expected scope need edits

## Output Required From Hermes
- branch name
- files changed
- summary of behavior change
- commands run and results
- risks
- screenshots/logs if relevant
- questions for human review
```

---

## 9. Research note schema

Research should not become a dumping ground.

```md
---
title:
topic:
status: inbox | accepted | rejected
created:
source_count:
projects: []
tags: []
---

# Research: Topic

## Question
What were we trying to answer?

## Keep
- Point 1 worth remembering.
- Point 2 worth remembering.
- Point 3 if needed.

## Decision
What should we do because of this research?

## Rejected
- Approach rejected and why.

## Applies To
- Global guideline?
- One project?
- Future project?

## Next Action
- Create/update guideline.
- Create/update project decision.
- Create task packet.
- Do nothing.

## Sources
- Link/source 1
- Link/source 2
```

Rule:

> If research produces no decision, no guideline, and no future action, it stays in `research/inbox/` and can be deleted later.

---

## 10. Hermes + Telegram workflow

Telegram should become the remote control center, not the place where architecture is invented from scratch.

### 10.1 Normal flow

```text
Human on Telegram:
"Yawnly: create task packet for no-internet handling in story generation. Read project context and global CMP/Supabase rules. Do not code yet."

Hermes:
Creates draft task packet and sends summary.

Human:
"Approved. Implement on branch hermes/yawnly-no-internet. Only touch allowed paths. Run compile. Send diff summary."

Hermes:
Checks out branch, edits files, runs verification, commits locally or prepares diff.

Human:
"Show changed files, risks, and anything you were unsure about."

Hermes:
Sends review report.

Human:
"Open PR" or "revise" or "stop."
```

### 10.2 Command levels

Hermes should support different task modes.

| Mode | Meaning |
|---|---|
| `research-only` | Search/read/summarize. No code edits. |
| `plan-only` | Create task packet/spec. No code edits. |
| `implement-local` | Edit allowed paths on a branch. No push. |
| `implement-pr` | Edit allowed paths, commit, push branch, open PR. |
| `review-only` | Review diff/branch and report findings. |
| `wiki-proposal` | Propose wiki updates in inbox. Do not alter canonical docs. |

Default mode should be `plan-only` unless explicitly upgraded.

### 10.3 Review report format

Hermes must report:

```md
## Summary

## Files Changed

## Verification

## Architecture Compliance

## Risks / Open Questions

## Suggested Wiki Updates
```

Suggested wiki updates go to `inbox/agent-proposals/`, not directly to canonical files.

---

## 11. Safety model

Text instructions are not enough. Do not rely only on "Hermes, stay in this folder."

### 11.1 Minimum safe setup

For early testing:

- run Hermes with only one project open
- use branch-only work
- no direct commits to `main`
- no direct pushes unless explicitly approved
- no access to day-job repos
- no raw secrets in wiki
- use API models for reasoning rather than running heavy local models during work hours

### 11.2 Better safe setup

Preferred once Hermes is working:

- separate macOS user or Docker/container backend
- mount only:
  - `llm-wiki`
  - one target repo
  - scratch directory
- restrict shell/browser toolsets by task mode
- use Telegram allowlists/pairing
- disable browser/GUI tools for background code tasks unless explicitly needed
- keep provider keys outside wiki

### 11.3 Branch rules

Hermes must:

- create branches like `hermes/<project>-<task>`
- never work directly on `main`
- never force push
- never change secrets
- never modify files outside task packet allowed paths
- stop if scope expands unexpectedly

---

## 12. Search strategy

Start simple:

- `index.md`
- good filenames
- frontmatter
- `rg`

Do not install `qmd` at launch.

Adopt `qmd` only if:

- useful pages exceed roughly 300-500
- `rg` fails to retrieve relevant notes
- Hermes needs MCP search across the wiki
- the wiki is already disciplined enough that semantic search will not amplify stale junk

If adopted, `qmd` is an index, not the source of truth.

---

## 13. Maintenance rules

### 13.1 Weekly, 15 minutes

- process `inbox/quick-capture/`
- promote or delete research inbox notes
- update one active project's `current-state.md`
- review Hermes wiki proposals

### 13.2 Monthly, 30 minutes

- check stale links
- check missing referenced files
- check docs over size caps
- archive dead task packets
- review global guidelines for drift

### 13.3 Quarterly, 1-2 hours

- prune stale pages
- review project decisions
- update linked guidelines for each active project
- archive or merge duplicated rules
- decide whether search tooling needs upgrade

No auto-archive at launch. Auto-flag only.

---

## 14. Bloat controls

### 14.1 Promotion question

Before adding a canonical page:

> Will I or an agent use this on another day?

If no, keep it in inbox or delete it.

### 14.2 Single-source rule

Each fact has one home:

- reusable rule → `guidelines/`
- project decision → `projects/<project>/decisions.md`
- project state → `projects/<project>/current-state.md`
- task execution details → `task-packets/`
- research conclusion → `research/accepted/`

Do not duplicate facts across all project docs. Link to the source.

### 14.3 Soft size limits

| File type | Soft cap |
|---|---:|
| `index.md` | 150 lines |
| guideline page | 300 lines |
| project `context.md` | 150 lines |
| project `decisions.md` | 300 lines before splitting |
| `current-state.md` | 100 lines |
| task packet | 150 lines |
| research accepted note | 120 lines |

If a file exceeds the cap, split or summarize.

---

## 15. What to extract from Yawnly first

Yawnly is the pilot project because it already shows the problem clearly.

### 15.1 Extract to global guidelines

From Yawnly docs, extract reusable rules:

```text
guidelines/frontend/compose-multiplatform.md
guidelines/frontend/compose-ui.md
guidelines/backend/supabase.md
guidelines/backend/supabase-edge-functions.md
guidelines/shared/api-contracts.md
guidelines/agents/spec-first-workflow.md
```

Examples of global CMP rules:

- shared code lives in `composeApp/src/commonMain`
- feature folder shape: `domain`, `data`, `presentation`, `di`
- ViewModel → UseCase → Repository interface → RepositoryImpl → ApiService
- domain models do not get `@Serializable`
- DTOs use `@Serializable`, `@SerialName`, and mapper to domain
- Root composable handles ViewModel, state collection, side effects, and navigation
- content composable is dumb: `state` + `onAction`
- Koin modules must be registered for relevant platforms
- core should not import feature types

Examples of global Supabase rules:

- secrets stay server-side
- mobile app uses anon key/user JWT only
- service-role logic belongs in Edge Functions
- migrations, DTOs, and function payloads must stay aligned
- read actual table schema before SQL changes
- RLS policy changes require explicit verification

### 15.2 Keep in Yawnly project folder

```text
projects/yawnly/context.md
projects/yawnly/decisions.md
projects/yawnly/current-state.md
projects/yawnly/repo-map.md
projects/yawnly/backlog.md
projects/yawnly/linked-guidelines.md
```

Yawnly-specific decisions to capture:

- Supabase is the product backend
- legacy Ktor/BaseRepository is retained skeleton code, not new product path
- no Room for Yawnly product data
- audio-only/story flow before 3D buddy
- mythology mode after launch/when scoped
- story library and retention features likely before heavy visual buddy work
- prompt generation uses structured prompts/entropy, not RAG
- PII placeholders in model-facing prompts
- subscription state should be server-authoritative

### 15.3 Fix stale references

Yawnly currently references docs that appear absent in the checkout:

- `plan/yawnly-cmp-implementation-plan.md`
- PRD files such as `plan/yawnly-prd-v3.md` / `v4`

The pilot should either:

- restore those files, or
- update project context to point to current real docs, or
- mark them as stale and not used by agents

---

## 16. Implementation phases

### Phase 0 — Review and approve v2 plan

Goal:

- agree on scope before building

Outputs:

- reviewed `llm-wiki-plan-v2.md`
- approved initial folder structure
- approved Yawnly pilot scope
- approved Hermes safety assumptions

Do not:

- build full wiki
- migrate all projects
- install qmd
- build quota automation

Success criteria:

- plan feels small enough to start
- first pilot project is Yawnly only
- global vs project-local boundary is clear

---

### Phase 1 — Create minimal wiki skeleton

Goal:

- create the smallest useful wiki repo

Create:

```text
llm-wiki/
├── index.md
├── README.md
├── AGENTS.md
├── HERMES.md
├── guidelines/
├── golden-examples/
├── projects/
│   ├── _template/
│   └── yawnly/
├── research/
│   ├── inbox/
│   └── accepted/
├── inbox/
└── meta/
```

Create schema files:

- `meta/page-schema.md`
- `meta/task-packet-schema.md`
- `meta/research-note-schema.md`

Create first global pages:

- `guidelines/frontend/compose-multiplatform.md`
- `guidelines/frontend/compose-ui.md`
- `guidelines/backend/supabase.md`
- `guidelines/agents/delegation-protocol.md`

Create first Yawnly pages:

- `projects/yawnly/context.md`
- `projects/yawnly/linked-guidelines.md`
- `projects/yawnly/decisions.md`
- `projects/yawnly/current-state.md`
- `projects/yawnly/repo-map.md`
- `projects/yawnly/task-packets/README.md`

Success criteria:

- agent can read `index.md` and know where Yawnly context lives
- Yawnly project page links to global CMP/Supabase rules
- duplicated Yawnly rules are reduced in future, not expanded

---

### Phase 2 — Link Yawnly repo to wiki

Goal:

- make Yawnly the first working project connected to the wiki

In Yawnly repo:

- keep `AGENTS.md` / `CLAUDE.md` short
- point agents to wiki pages
- remove or reduce repeated generic rules over time
- keep only project-specific gotchas in repo files

Example Yawnly repo bridge:

```md
# Yawnly Agent Context

Primary external context:
- `/path/to/llm-wiki/projects/yawnly/context.md`
- `/path/to/llm-wiki/projects/yawnly/linked-guidelines.md`

Before implementation:
1. Read project context.
2. Read linked guidelines relevant to the task.
3. Use a task packet for non-trivial work.
4. Do not code until task packet is approved.
```

Success criteria:

- Yawnly agents no longer need to infer architecture from scattered docs
- missing/stale references are called out
- Yawnly docs become thinner over time

---

### Phase 3 — Set up Hermes locally with safe Yawnly pilot

Goal:

- make Hermes controllable through Telegram for one repo

Setup:

- install and configure Hermes
- configure model provider(s)
- configure Telegram gateway
- connect Hermes to:
  - `llm-wiki`
  - Yawnly repo only
  - scratch directory
- run with restricted access if possible

Preferred safety:

- separate macOS user or Docker/container backend
- branch-only work
- no direct `main`
- no unapproved pushes
- no day-job repo access
- browser/GUI disabled for code tasks unless needed

First Hermes tasks:

1. `research-only`: summarize Yawnly repo map using wiki context
2. `plan-only`: create a task packet for a small Yawnly fix
3. `implement-local`: implement one low-risk task on a branch
4. `review-only`: ask Hermes to review its own diff against guidelines

Success criteria:

- Telegram command reaches Hermes
- Hermes reads wiki context
- Hermes works only in Yawnly
- Hermes creates a branch
- Hermes sends useful review summary
- human can approve/reject from Telegram

---

### Phase 4 — Prove the task packet loop

Goal:

- validate the core workflow before adding more projects

Pilot task candidates:

- update stale Yawnly doc references
- create no-code repo map
- implement a tiny UI consistency fix
- write or improve one test
- create a task packet for no-internet handling without implementation

Workflow:

1. Human asks Hermes to create task packet.
2. Human approves.
3. Hermes implements in branch.
4. Hermes runs verification.
5. Hermes sends review report.
6. Human reviews diff.
7. Hermes proposes wiki updates to inbox.
8. Human promotes only durable lessons.

Success criteria:

- human involvement is mostly approval/review
- no broad unexpected diffs
- agent follows allowed paths
- useful lessons are captured without bloating canonical docs

---

### Phase 5 — Add remaining CMP projects one by one

Goal:

- reuse CMP guidelines across all existing CMP apps

For each CMP project:

1. create `projects/<name>/context.md`
2. create `projects/<name>/linked-guidelines.md`
3. create `projects/<name>/decisions.md`
4. create `projects/<name>/current-state.md`
5. add project bridge file in repo
6. run one `plan-only` Hermes task
7. run one low-risk `implement-local` Hermes task

Do not bulk migrate everything.

Success criteria:

- each project has only project-specific docs
- all CMP projects reuse same global CMP rules
- project differences are explicit

---

### Phase 6 — Add backend standards

Goal:

- prepare for Spring backend and new FastAPI + Supabase project

Create/refine:

- `guidelines/backend/fastapi.md`
- `guidelines/backend/fastapi-supabase.md`
- `guidelines/backend/spring-kotlin.md`
- `guidelines/shared/auth.md`
- `guidelines/shared/api-contracts.md`
- `guidelines/shared/error-handling.md`

Create golden examples:

- FastAPI auth middleware
- FastAPI route/service/repository structure
- Supabase JWT verification
- Spring controller/service/repository pattern
- standard error response shape

Success criteria:

- new FastAPI project can start from explicit standards
- Hermes can research backend choices and distill conclusions
- backend patterns do not get invented repeatedly per project

---

### Phase 7 — Add research workflow

Goal:

- make research useful without storing huge dumps

Workflow:

1. capture raw research in `research/inbox/`
2. distill to `Keep / Decision / Rejected / Next Action`
3. promote only durable conclusions to:
   - guideline
   - project decision
   - task packet
   - accepted research note

Example use:

```text
"Hermes, research FastAPI + Supabase auth for the new CMP backend.
Return only: keep points, recommended approach, rejected approaches, and first task packet.
Do not write code."
```

Success criteria:

- research produces decisions
- accepted notes stay short
- global guidelines improve over time

---

### Phase 8 — Add maintenance automation

Goal:

- prevent wiki drift without building a custom platform

Add simple scripts/CI:

- markdown lint
- link check
- missing referenced file check
- page size check
- stale page report
- inbox age report
- task packet status report

Do not:

- auto-delete
- auto-archive
- auto-promote agent notes

Success criteria:

- monthly drift report is useful
- broken links and stale references are visible
- maintenance remains under 30 minutes/month

---

### Phase 9 — Optional search upgrade

Goal:

- add `qmd` only if simple search stops working

Adopt when:

- page count is high
- search misses useful notes
- Hermes needs MCP wiki search

Rules:

- exclude archives, logs, rejected research, generated reports
- keep Markdown as canonical source
- do not trust semantic search over project context and task packets

---

### Phase 10 — Ops/vendor/quota tracking

Goal:

- add ops tracking only after engineering workflow is stable

Add:

- `ops/vendors/`
- `ops/quotas/`
- `ops/servers/`

Start with only vendors that can cost money soon:

- Supabase
- OpenRouter/model providers
- Modal/Vercel/Cloudflare if used
- app store subscriptions/accounts

Do not build cross-vendor automation until manual tracking is painful.

---

## 17. Anti-patterns to avoid

1. Building the whole wiki before proving Yawnly pilot.
2. Letting Hermes write canonical guidelines automatically.
3. Storing every research source forever.
4. Making global decisions out of project-only decisions.
5. Duplicating global CMP rules in every project.
6. Letting task packets become giant specs.
7. Running Hermes unrestricted on the same Mac user with access to all repos.
8. Installing qmd/vector search before plain search fails.
9. Treating Telegram chat history as source of truth.
10. Treating Hermes memory as source of truth.
11. Using the wiki as a product backlog replacement.
12. Migrating all projects in one pass.

---

## 18. Immediate next actions

If this v2 plan is approved:

1. Create `~/dev/llm-wiki` or another chosen repo location.
2. Create the minimal skeleton from Phase 1.
3. Extract first CMP guideline from Yawnly docs.
4. Create `projects/yawnly/context.md`.
5. Create `projects/yawnly/decisions.md` with 5-10 important decisions.
6. Create one Yawnly task packet.
7. Set up Hermes with Telegram and access only to wiki + Yawnly.
8. Run one `plan-only` task.
9. Run one low-risk `implement-local` task.
10. Review whether this actually reduced your involvement.

Only after that should more projects be added.

---

## 19. Success metrics

This system is working if:

- you repeat fewer architecture explanations
- Hermes asks fewer basic project questions
- agents produce smaller diffs
- task review happens through branch reports, not line-by-line babysitting
- product decisions are easy to find
- research turns into 2-5 remembered conclusions
- one CMP guideline improves all CMP projects
- new projects start faster because standards already exist
- Telegram can safely trigger work without giving Hermes the whole machine

This system is failing if:

- you spend more time maintaining the wiki than coding
- agents still read everything
- task packets become massive
- project decisions are duplicated globally
- Hermes changes files outside allowed scope
- accepted research becomes long unreadable dumps
- you feel pressure to keep every note forever

---

## 20. Final recommendation

Build the system in this order:

1. Wiki skeleton.
2. Yawnly as the first pilot.
3. One global CMP guideline.
4. One global Supabase guideline.
5. Yawnly project context and decisions.
6. One task packet.
7. Hermes + Telegram safe execution.
8. Prove one complete loop.
9. Add other projects only after the loop works.

This keeps the wiki simple while still solving the real problem: giving Hermes enough durable context and boundaries to work across your projects without forcing you to explain everything every time.
