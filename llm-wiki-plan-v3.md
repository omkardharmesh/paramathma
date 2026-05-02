# Personal LLM Wiki v3 — Final Hermes Control Layer Plan

**Status:** Final draft for approval  
**Date:** 2026-05-02  
**Goal:** Create a small, durable, Markdown-based engineering wiki that helps Hermes and coding agents work across projects with less repeated explanation, smaller diffs, and safer Telegram-driven delegation.

---

## 1. Final Mental Model

Hermes is the active operating system.

The wiki is the stable constitution.

Hermes already owns a lot:

- memory
- searchable sessions
- skills
- cron
- Telegram/gateway control
- profiles
- approval policies
- Docker/backend isolation
- session logs

So the wiki must not rebuild Hermes.

The wiki only owns stable truths:

- global FE/BE engineering guidelines
- project context
- project decisions
- golden examples
- distilled research conclusions
- task packets that define safe work boundaries

Everything else should stay in Hermes, Git, GitHub/Linear, or the project repos.

---

## 2. Problem This Solves

You are managing:

- 4 Compose Multiplatform apps
- Yawnly with Supabase + Edge Functions
- 1 Spring backend
- 1 new CMP project planned with FastAPI + Supabase
- future web, DevOps, and self-hosted work

The pain is not "I need more docs."

The pain is:

- agents forget your architecture
- agents mix patterns across projects
- you repeat the same FE/BE rules
- product decisions get buried in scattered docs
- research produces too much noise and too few reusable conclusions
- you want Hermes to work from Telegram while you are busy
- you still need review control and safe project boundaries

The solution is a small central wiki plus Hermes task packets.

---

## 3. Community Patterns Kept

This plan keeps the strongest ideas from the LLM wiki community, but narrows them for your workload.

References to keep in mind:

- [Karpathy LLM-Wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [DAIR.AI breakdown of Karpathy pattern](https://academy.dair.ai/blog/llm-knowledge-bases-karpathy)
- [AGENTS.md](https://agents.md/)
- [OpenAI Codex AGENTS.md docs](https://developers.openai.com/codex/guides/agents-md/)
- [qmd](https://github.com/tobi/qmd)
- [llms.txt](https://llmstxt.org/)
- [SamurAIGPT/llm-wiki-agent](https://github.com/SamurAIGPT/llm-wiki-agent)
- [Ar9av/obsidian-wiki](https://github.com/Ar9av/obsidian-wiki)
- [Hermes Atlas guide](https://hermesatlas.com/guide/)

Patterns kept:

- Markdown-first, git-backed source of truth.
- `index.md` first; never recursive-load the wiki.
- Search starts with filenames, `rg`, and frontmatter.
- Raw capture is separate from canonical knowledge.
- Agents propose updates; humans promote truth.
- Small pages beat giant docs.
- Link to external docs; store only your opinionated take, gotchas, and decisions.
- qmd/MCP search comes later only if simple search fails.

What this plan changes:

- It is not a general "second brain."
- It is not an autonomous memory system.
- It is not a central task manager.
- It is not a custom RAG system.
- It is specifically a Hermes/coding-agent control layer.

Review rule:

> Add only what reduces repeated explanation, reduces review burden, or improves Hermes task safety.

---

## 4. What The Wiki Owns vs Hermes Owns


| Concern                         | Owner                   |
| ------------------------------- | ----------------------- |
| Stable engineering guidelines   | Wiki                    |
| Project context and deviations  | Wiki                    |
| Project decisions               | Wiki                    |
| Reusable code examples/patterns | Wiki                    |
| Research conclusions            | Wiki                    |
| Task boundaries                 | Wiki task packets       |
| Execution                       | Hermes                  |
| Telegram control                | Hermes                  |
| Cron/scheduled jobs             | Hermes                  |
| Session logs                    | Hermes                  |
| Learned procedures              | Hermes skills           |
| Short-term memory               | Hermes memory           |
| Source code truth               | Project repo            |
| Large backlog/issue tracking    | GitHub/Linear, not wiki |


This boundary prevents v3 from becoming v1 again.

---

## 5. Final Folder Structure

Launch structure:

```text
MyWiki/
├── index.md
├── README.md
├── AGENTS.md
├── HERMES.md
│
├── guidelines/
│   ├── frontend/
│   │   ├── compose-multiplatform.md
│   │   └── compose-ui.md
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
│       └── hermes-delegation.md
│
├── golden-examples/
│   ├── cmp/
│   ├── supabase/
│   ├── fastapi/
│   └── spring/
│
├── projects/
│   ├── _template/
│   └── yawnly/
│       ├── context.md
│       ├── linked-guidelines.md
│       ├── decisions.md
│       ├── current-state.md
│       ├── repo-map.md
│       ├── backlog.md
│       └── task-packets/
│
├── research/
│   ├── inbox/
│   ├── accepted/
│   └── rejected/
│
├── inbox/
│   ├── quick-capture/
│   ├── hermes-proposals/
│   └── claude-proposals/
│
└── meta/
    ├── page-schema.md
    ├── task-packet-schema.md
    ├── research-note-schema.md
    ├── maintenance-checklist.md
    └── wikiignore.md
```

Do not create `ops/`, `vendors/`, `runbooks/`, `postmortems/`, `prompts/`, `glossary.md`, or global `TASKS.md` at launch.

Add them only when the trigger appears:


| Add later                    | Trigger                                                     |
| ---------------------------- | ----------------------------------------------------------- |
| `ops/vendors/`               | First real vendor lock-in/cost question                     |
| `ops/quotas/`                | First real free-tier/billing risk                           |
| `runbooks/`                  | First repeated operational procedure                        |
| `postmortems/`               | First production incident or serious regression             |
| `glossary.md`                | Agents hallucinate private terms                            |
| `prompts/`                   | Saved prompts become repeatedly useful outside task packets |
| `TASKS.md` / in-flight board | Multiple agents run overlapping work                        |
| `qmd`                        | `rg`/index search stops being enough                        |


---

## 6. Page Schema

Every canonical page should use a small frontmatter schema.

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

Notes:

- `last_verified` means a human or approved agent pass confirmed the content is still true.
- Do not update frontmatter on every agent read. That creates noisy commits.
- If usage tracking becomes needed later, use `meta/page-usage.tsv`, not per-page edits.
- `summary` is important because agents can scan summaries cheaply before reading full pages.

---

## 7. Agent Read Protocol

Agents should read in this order:

1. `index.md`
2. `projects/<project>/context.md`
3. `projects/<project>/linked-guidelines.md`
4. only task-relevant guideline pages
5. only task-relevant golden examples
6. current task packet

Agents must not recursively read:

- all `projects/`
- all `research/`
- all `inbox/`
- all `golden-examples/`
- generated graph reports
- archived/stale files

`index.md` is the map, not the manual.

---

## 8. Global Guidelines

Global guidelines are reusable standards outside all projects.

Examples:

```text
guidelines/frontend/compose-multiplatform.md
guidelines/frontend/compose-ui.md
guidelines/backend/supabase.md
guidelines/backend/supabase-edge-functions.md
guidelines/backend/fastapi.md
guidelines/backend/spring-kotlin.md
guidelines/shared/api-contracts.md
guidelines/shared/error-handling.md
```

Rules for guidelines:

- Store reusable rules only.
- Keep project-specific deviations out.
- Prefer "do/don't" and checklists over essays.
- Link to external docs instead of copying them.
- Link to golden examples when concrete code matters.

Examples of global CMP rules:

- shared code lives in `composeApp/src/commonMain`
- feature structure uses domain/data/presentation/di
- ViewModel talks to use cases, not transport directly
- domain models do not use `@Serializable`
- DTOs use `@Serializable` and map to domain
- Root composable handles ViewModel, state, side effects, navigation
- content composable is dumb: `state` + `onAction`
- Koin module registration must include all relevant platforms
- core must not import feature types

---

## 9. Golden Examples

Guidelines say the rule.

Golden examples show the pattern.

Examples:

```text
golden-examples/cmp/screen-root-content-pattern.md
golden-examples/cmp/clean-feature-module.md
golden-examples/cmp/supabase-repository-pattern.md
golden-examples/supabase/edge-function-pattern.md
golden-examples/fastapi/auth-middleware.md
golden-examples/spring/controller-service-repository.md
```

Each golden example should include:

- when to use it
- source project/file it came from
- minimal code shape or file map
- common mistakes
- verification command if relevant

Do not store huge code dumps. Link to the repo/file when possible.

---

## 10. Project Folder Contract

Each project gets a small folder in `projects/<name>/`.

### 10.1 `context.md`

Purpose: tell agents what the project is.

Include:

- product summary
- repo path
- stack
- platform targets
- backend/services
- build/test commands
- current release status
- hard constraints
- what not to touch
- project-specific deviations

### 10.2 `linked-guidelines.md`

Purpose: declare exactly which global rules apply.

Example:

```md
# Yawnly Linked Guidelines

Read for CMP work:
- `guidelines/frontend/compose-multiplatform.md`
- `guidelines/frontend/compose-ui.md`

Read for backend work:
- `guidelines/backend/supabase.md`
- `guidelines/backend/supabase-edge-functions.md`

Use examples:
- `golden-examples/cmp/screen-root-content-pattern.md`
- `golden-examples/cmp/supabase-repository-pattern.md`
```

### 10.3 `decisions.md`

Purpose: project-local decision memory.

Most decisions belong here, not globally.

Format:

```md
## D001: Ship Audio-Only Before 3D Buddy

Status: accepted
Date: 2026-05-02
Scope: Yawnly

Decision:
Ship the audio/story flow before building the 3D/Lottie buddy.

Why:
The buddy adds animation and rendering complexity. Validate payment, retention, and story quality first.

Rejected:
- Full 3D animation pipeline before v1
- Buddy as launch blocker

Revisit:
After retention/payment are validated or around 500 paying users.
```

### 10.4 `current-state.md`

Purpose: short live snapshot.

Include:

- currently working
- in progress
- blocked/broken
- release risks
- next recommended work

Keep it under 100 lines.

### 10.5 `repo-map.md`

Purpose: orient agents without scanning the whole repo.

Include:

- modules
- important directories
- backend locations
- test/build commands
- generated folders to avoid
- stale/missing docs to ignore

### 10.6 `backlog.md`

Purpose: tiny project-local priority view.

Allowed sections:

- release-critical
- next
- later
- parked

This is not a full product backlog. If it grows beyond 100-150 lines, move to GitHub/Linear or split into task packets.

### 10.7 `task-packets/`

Purpose: exact Hermes work orders.

This is the most important project folder.

---

## 11. Task Packet Schema

Hermes should execute task packets, not vague wishes.

```yaml
---
id: yawnly-YYYYMMDD-short-name
project: yawnly
repo_path: /Users/eloelo/Downloads/Yawnly
status: draft | approved | running | review | done | cancelled
mode: research-only | plan-only | implement-local | implement-pr | review-only | wiki-proposal
risk: low | medium | high
created: YYYY-MM-DD
owner: human
executor: hermes
---
```

```md
# Task: Short Name

## Goal
One paragraph describing what should change.

## Why
Product or technical reason.

## End Goal
The precise outcome Hermes should optimize for.

## Output Format
The exact format expected from Hermes: report, branch, PR, task packet, diff summary, or wiki proposal.

## Recommended Approach
The preferred sequence of work. Hermes may propose a safer alternative before execution.

## Common Failure Handling
Known traps and what Hermes must do when they appear.

## Allowed Paths
- `composeApp/src/commonMain/...`
- `supabase/functions/...`

Hermes must not edit outside these paths.

## Required Context
- `projects/yawnly/context.md`
- `projects/yawnly/linked-guidelines.md`
- relevant guideline
- relevant golden example

## Reference Files In Repo
- `path/to/good/example/File.kt`

## Non-Goals
- Do not refactor unrelated files.
- Do not change public contracts unless listed.
- Do not touch secrets.

## Implementation Plan
1. Step one.
2. Step two.
3. Step three.

## Verification
Commands:
- `rtk ./gradlew :composeApp:compileKotlinAndroid`
- `rtk ./gradlew :androidApp:assembleDebug`

## Stop Conditions
Stop and ask human if:
- task needs paths outside Allowed Paths
- dependency changes are needed
- secrets are needed
- build requires unrelated fixes
- architecture differs from linked guidelines
- more than 3 unexpected files need edits

## Output Required
- branch name
- files changed
- summary
- commands run and results
- risks/open questions
- suggested wiki updates, if any
```

Default mode is `plan-only`.

Human must explicitly approve `implement-local` or `implement-pr`.

---

## 12. Hermes Delegation Modes


| Mode              | Meaning                                 |
| ----------------- | --------------------------------------- |
| `research-only`   | Search/read/summarize. No code edits.   |
| `plan-only`       | Create task packet/spec. No code edits. |
| `implement-local` | Edit allowed paths on branch. No push.  |
| `implement-pr`    | Edit, commit, push branch, open PR.     |
| `review-only`     | Review branch/diff and report findings. |
| `wiki-proposal`   | Suggest wiki updates in inbox only.     |


Hermes report format:

```md
## Summary
## Files Changed
## Verification
## Architecture Compliance
## Risks / Open Questions
## Suggested Wiki Updates
```

Suggested wiki updates go to `inbox/hermes-proposals/`.

They must be reviewed weekly. Delete or archive proposals older than 14 days unless they are actively being promoted.

---

## 13. Research Workflow

Research should produce conclusions, not dumps.

```yaml
---
title:
topic:
status: inbox | accepted | rejected
created:
projects: []
tags: []
---
```

```md
# Research: Topic

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
- Link/source
```

If research has no decision, no reusable conclusion, and no next action, it should not become canonical.

Telegram capture path:

```text
You -> Hermes:
"Capture this research thought for FastAPI Supabase auth. Put it in research inbox. Extract only Keep / Decision / Rejected / Next Action."
```

No separate mobile capture system at launch. Telegram is enough.

---

## 14. Agent Write Lanes

Human owns canonical truth.

Agents own proposal lanes.


| Actor         | Can write without explicit approval                                         | Needs explicit approval                 |
| ------------- | --------------------------------------------------------------------------- | --------------------------------------- |
| Human         | anywhere                                                                    | none                                    |
| Hermes        | `inbox/hermes-proposals/`, task reports, approved task branch               | canonical guidelines, project decisions |
| Claude/Cursor | `inbox/claude-proposals/`, task reports, current working repo when approved | canonical guidelines, project decisions |


If Hermes learns a useful pattern, it may:

- create/update its own Hermes skill
- propose a wiki update
- create a task packet

It must not silently rewrite canonical project decisions or global guidelines.

---

## 15. Secrets Convention

Never store raw secrets in the wiki.

Allowed:

- environment variable names
- Supabase secret names
- 1Password references such as `op://vault/item/field`
- Bitwarden item names or IDs
- setup instructions that say where a secret lives

Not allowed:

- API keys
- tokens
- JWTs
- service role keys
- private keys
- `.env` contents

If sensitive billing/subscription details are eventually stored, use encryption for only that narrow folder/file. Do not encrypt the whole wiki.

---

## 16. Hermes Safety Setup

Minimum launch safety:

- Hermes sees only `MyWiki`, Yawnly, and a scratch directory.
- Hermes works on branches, never directly on `main`.
- No unapproved push.
- No force push.
- No day-job repos mounted or reachable.
- No secrets mounted unless a task explicitly requires them.
- Browser/GUI tools disabled for background coding unless needed.
- Approval policies enabled for destructive operations.

Preferred setup:

- Docker backend or separate macOS user.
- Separate Hermes profile for personal projects.
- Telegram allowlist uses numeric Telegram user ID.
- Frontier/good tool-calling model for multi-step code tasks.
- Cheap/local models only for low-risk summarization/research.

Branch rule:

```text
hermes/<project>-<task>
```

Hermes stops if the task wants files outside the task packet's Allowed Paths.

---

## 17. Yawnly Pilot

Yawnly is the first project because it already shows the problem:

- many docs exist
- rules are duplicated
- product decisions are scattered
- some referenced docs are missing or stale
- CMP + Supabase patterns are reusable across future projects

### 17.1 Extract to global guidelines

From Yawnly, extract reusable rules into:

```text
guidelines/frontend/compose-multiplatform.md
guidelines/frontend/compose-ui.md
guidelines/backend/supabase.md
guidelines/backend/supabase-edge-functions.md
guidelines/shared/api-contracts.md
```

### 17.2 Keep project-local

Yawnly-specific:

- Yawnly is an Indian kids bedtime story app.
- Supabase is the product backend.
- Ktor/BaseRepository is legacy skeleton, not new product path.
- Product data is online-only; no Room for Yawnly product data.
- Audio-only/story flow before 3D/Lottie buddy.
- Prompt generation uses structured prompts/entropy, not RAG.
- PII placeholders for model-facing prompts.
- Subscription truth should be server-authoritative.
- Story Library and retention features likely before heavy visual buddy work.

### 17.3 Fix stale input before pilot

Before Hermes works on Yawnly, fix or mark stale references such as:

- missing `plan/yawnly-cmp-implementation-plan.md`
- missing or stale PRD references
- outdated repo/module descriptions
- outdated migration counts

Do not pilot Hermes on bad documentation.

---

## 18. Implementation Plan

### Phase 0 — Approve v3

Goal:

- agree this is final plan
- stop iterating on planning
- move to setup

Success:

- `llm-wiki-plan-v3.md` approved
- Yawnly confirmed as first pilot
- Hermes safety assumptions accepted

---

### Phase 1 — Minimal Wiki + Yawnly Context

Create only:

```text
MyWiki/
├── index.md
├── README.md
├── AGENTS.md
├── HERMES.md
├── guidelines/
├── golden-examples/
├── projects/_template/
├── projects/yawnly/
├── research/
├── inbox/
└── meta/
```

Create:

- `meta/page-schema.md`
- `meta/task-packet-schema.md`
- `meta/research-note-schema.md`
- `guidelines/frontend/compose-multiplatform.md`
- `guidelines/frontend/compose-ui.md`
- `guidelines/backend/supabase.md`
- `guidelines/agents/hermes-delegation.md`
- `projects/yawnly/context.md`
- `projects/yawnly/linked-guidelines.md`
- `projects/yawnly/decisions.md`
- `projects/yawnly/current-state.md`
- `projects/yawnly/repo-map.md`
- `projects/yawnly/backlog.md`
- `projects/yawnly/task-packets/README.md`

Also in Phase 1:

- fix or mark stale Yawnly doc references
- add 5-10 important Yawnly decisions
- create one small Yawnly task packet

Success:

- agent can start from `index.md`
- Yawnly project context is clear
- global CMP/Supabase rules are reusable
- stale Yawnly references are not trusted

---

### Phase 2 — Link Yawnly Repo

Update Yawnly agent bridge files to point to the wiki.

Yawnly repo files should become thin:

```md
Read:
- `/path/to/MyWiki/projects/yawnly/context.md`
- `/path/to/MyWiki/projects/yawnly/linked-guidelines.md`

For non-trivial work:
- use a task packet
- do not code until task packet is approved
```

Do not delete useful repo docs immediately. Thin them over time as global rules move to the wiki.

Success:

- agents no longer rely on scattered Yawnly docs first
- project-specific exceptions remain visible
- generic CMP/Supabase rules live globally

---

### Phase 3 — Set Up Hermes + Telegram For Yawnly Only

Setup:

- install Hermes
- configure model provider
- configure Telegram gateway
- enable approval policies
- run in Docker/separate user if practical
- allow access only to:
  - `MyWiki`
  - Yawnly repo
  - scratch directory

First commands:

1. `research-only`: summarize Yawnly repo map using wiki context
2. `plan-only`: create a task packet for one tiny Yawnly task
3. `implement-local`: implement the approved tiny task on a branch
4. `review-only`: review its own diff against linked guidelines

Success:

- Telegram reaches Hermes
- Hermes reads wiki context
- Hermes works only in Yawnly
- Hermes creates branch
- Hermes returns useful review report
- human approves/rejects from Telegram

---

### Phase 4 — Prove One Complete Loop

Goal:

- prove the system saves time before adding projects

Loop:

1. Human sends request through Telegram.
2. Hermes creates task packet.
3. Human approves.
4. Hermes implements on branch.
5. Hermes runs verification.
6. Hermes sends report.
7. Human reviews diff.
8. Hermes proposes wiki updates to inbox.
9. Human promotes or deletes proposals within 14 days.

Good pilot tasks:

- fix stale Yawnly doc references
- update repo map
- implement tiny UI consistency fix
- write one small test
- create no-internet handling task packet without coding

Success:

- less human hand-holding
- smaller diffs
- no path violations
- useful report
- useful wiki proposal or no proposal

---

### Phase 5 — Expand Only After Proof

After Phase 4 works:

- add other CMP projects one by one
- add FastAPI/Supabase guidelines for new backend
- add Spring backend guideline
- add more golden examples from real projects
- add research workflow as needed

Do not bulk migrate.

For each new project:

1. create project folder
2. link global guidelines
3. add decisions/current-state/repo-map
4. create one task packet
5. run one `plan-only` task
6. run one low-risk `implement-local` task

---

## 19. Maintenance

Weekly, 15 minutes:

- review `inbox/quick-capture/`
- review `inbox/hermes-proposals/`
- promote or delete proposals older than 14 days
- update active project `current-state.md` if needed

Monthly, 30 minutes:

- check broken links
- check stale references
- check page sizes
- check task packets stuck in `review` or `running`

Quarterly, 1-2 hours:

- calendar this from day one
- prune stale docs
- verify canonical guidelines
- review project decisions
- archive dead research
- decide if qmd/search upgrade is needed

Hard rule:

> If quarterly prune is skipped twice, pause expansion and clean the wiki before adding projects or automation.

---

## 20. Size Caps

Soft caps:


| File type              | Soft cap  |
| ---------------------- | --------- |
| `index.md`             | 150 lines |
| guideline              | 300 lines |
| project `context.md`   | 150 lines |
| project `decisions.md` | 300 lines |
| `current-state.md`     | 100 lines |
| task packet            | 150 lines |
| accepted research note | 120 lines |


Enforcement:

- Manual at launch.
- Add script later if size drift appears.
- Script should warn at soft cap and fail only at hard cap.

Do not build CI before the Yawnly Hermes loop works.

---

## 21. Search

Start with:

- `index.md`
- clear filenames
- frontmatter summaries
- `rg`

Do not install qmd at launch.

Add qmd only when:

- useful pages exceed roughly 300-500
- `rg` misses useful notes
- Hermes needs MCP wiki search
- the wiki is disciplined enough that semantic search will not amplify stale junk

qmd is an index, not the source of truth.

---

## 22. What Not To Add Now

Do not add at launch:

- central `TASKS.md`
- prompts library
- ops/vendor tracker
- quota automation
- custom vector DB
- qmd
- Notion backend
- auto-archive
- custom Hermes memory replacement
- heavy multi-agent ownership system
- mobile capture app workflow
- wiki publishing site

All of these are later triggers, not launch requirements.

---

## 23. Success Metrics

This is working if:

- you repeat fewer architecture explanations
- Hermes asks fewer basic project questions
- diffs are smaller
- task review happens through reports and branches
- Yawnly decisions are easy to find
- research turns into 2-5 useful points
- one CMP guideline helps multiple projects
- Telegram can safely trigger work
- Hermes does not touch files outside allowed paths

This is failing if:

- wiki maintenance becomes another job
- task packets become huge specs
- Hermes still reads everything
- project decisions move global by default
- inbox becomes a graveyard
- accepted research becomes long dumps
- you feel pressure to store everything

---

## 24. Final Recommendation

Approve v3 and build only the first loop:

1. Create minimal wiki.
2. Extract CMP/Supabase rules from Yawnly.
3. Create Yawnly project context and decisions.
4. Fix stale Yawnly references.
5. Create one Yawnly task packet.
6. Set up Hermes + Telegram with Yawnly-only access.
7. Run one complete task-packet loop.
8. Expand only after the loop saves time.

The wiki should stay boring.

Hermes should do the heavy lifting.

The task packet is the contract between them.