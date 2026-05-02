# Requirements Document — LLM Wiki v3 Setup + Yawnly Pilot

## Enforcement Contract

1. This spec is planning-only. It defines future documentation, wiki setup, and Hermes workflow setup. It does not authorize code implementation in Yawnly or any other project.
2. Implementation must not start until `llm-wiki-v3-setup-requirements.md`, `llm-wiki-v3-setup-design.md`, and `llm-wiki-v3-setup-tasks.md` have zero `P0/P1` red-team findings.
3. Spec truth lives in these three files plus `llm-wiki-plan-v3.md`, not chat memory.
4. Any risky claim is labeled `Verified` or `Out of Scope`.
5. Acceptance criteria use concrete behavior. If a claim is unverified, it is not an acceptance criterion.

---

## Introduction

This spec defines the first implementation pass for the Personal LLM Wiki v3 system. The system is a small Markdown wiki that acts as a stable control layer for Hermes and coding agents.

The first pilot project is Yawnly, a Compose Multiplatform app in `/Users/eloelo/Downloads/Yawnly`. The pilot will extract reusable CMP/Supabase guidance from Yawnly into global wiki guidelines, preserve Yawnly-specific decisions under `projects/yawnly/`, and create one Hermes task packet to prove the workflow.

This spec intentionally excludes Yawnly source-code changes.

---

## Verified Inputs

- `llm-wiki-plan-v3.md` — Verified 2026-05-02, source: `/Users/eloelo/Downloads/MyWiki/llm-wiki-plan-v3.md`.
- Yawnly spec workflow templates — Verified 2026-05-02, sources:
  - `/Users/eloelo/Downloads/Yawnly/.spec-workflow/user-templates/requirements-template.md`
  - `/Users/eloelo/Downloads/Yawnly/.spec-workflow/user-templates/design-template.md`
  - `/Users/eloelo/Downloads/Yawnly/.spec-workflow/user-templates/tasks-template.md`
  - `/Users/eloelo/Downloads/Yawnly/.spec-workflow/steering/tech.md`
- Yawnly agent docs — Verified 2026-05-02, sources:
  - `/Users/eloelo/Downloads/Yawnly/AGENTS.md`
  - `/Users/eloelo/Downloads/Yawnly/CLAUDE.md`
  - `/Users/eloelo/Downloads/Yawnly/docs/agent-architecture.md`
  - `/Users/eloelo/Downloads/Yawnly/docs/yawnly-ui-ux-polish-guide.md`
  - `/Users/eloelo/Downloads/Yawnly/docs/codebase-review.md`
  - `/Users/eloelo/Downloads/Yawnly/DESIGN.md`
- Yawnly product/planning docs — Verified 2026-05-02, sources:
  - `/Users/eloelo/Downloads/Yawnly/plan/v2/pending/ideation.md`
  - `/Users/eloelo/Downloads/Yawnly/plan/v2/pending/release-pending.md`
  - `/Users/eloelo/Downloads/Yawnly/plan/otp_integration.md`
  - `/Users/eloelo/Downloads/Yawnly/plan/v2/komodo/komodo_supabase_wiring_map.md`
  - `/Users/eloelo/Downloads/Yawnly/plan/v2/komodo/supabase_iap_server_impl_spec.md`
- Hermes Atlas guide — Verified 2026-05-02, source: `https://hermesatlas.com/guide/`.

---

## Alignment With Goal

The goal is not to build a large second brain. The goal is to make Hermes and coding agents more reliable across several projects by giving them:

- reusable global engineering rules
- project-specific context and decisions
- exact task packets with allowed paths and stop conditions
- a Telegram-controlled delegation flow

Yawnly is the first pilot because it already contains enough documentation and complexity to prove whether the system reduces repeated explanation and review burden.

---

## Requirements

### Requirement 1 — Minimal Wiki Skeleton

**User Story:** As the project owner, I want a minimal Markdown wiki skeleton, so that Hermes and coding agents have a stable place to read global guidelines and project context.

#### Acceptance Criteria

1. WHEN Phase 1 setup is complete THEN the wiki SHALL exist at `/Users/eloelo/Downloads/MyWiki`.
2. WHEN the wiki skeleton is created THEN it SHALL contain only the launch directories listed in `llm-wiki-plan-v3.md`: `guidelines/`, `golden-examples/`, `projects/`, `research/`, `inbox/`, and `meta/`.
3. WHEN the wiki skeleton is created THEN it SHALL contain `index.md`, `README.md`, `AGENTS.md`, and `HERMES.md`.
4. WHEN an agent reads `index.md` THEN it SHALL see the first-read order, active project list, and links to Yawnly context.
5. WHEN an agent reads `meta/wikiignore.md` THEN it SHALL see paths that agents must not bulk-load.

### Requirement 2 — Page and Schema Contracts

**User Story:** As the project owner, I want explicit page schemas, so that agents produce consistent wiki pages without over-templating.

#### Acceptance Criteria

1. WHEN canonical pages are created THEN they SHALL use the frontmatter fields defined in `meta/page-schema.md`.
2. WHEN project files are created THEN they SHALL use minimal frontmatter: `title`, `summary`, `type`, `status`, `updated`.
3. WHEN task packets are created THEN they SHALL use the schema from `meta/task-packet-schema.md`.
4. WHEN research notes are created THEN they SHALL use the schema from `meta/research-note-schema.md`.
5. WHEN a page is read by an agent THEN the page frontmatter SHALL NOT be modified only to record the read.

### Requirement 3 — Global Guidelines Extracted From Yawnly

**User Story:** As a solo developer with several CMP apps, I want reusable Yawnly-derived CMP and Supabase rules moved into global guidelines, so that future projects can reference the same standards without copying Yawnly docs.

#### Acceptance Criteria

1. WHEN global CMP rules are extracted THEN `guidelines/frontend/compose-multiplatform.md` SHALL include feature folder structure, layer direction, ViewModel/use-case/repository boundaries, DTO/domain separation, Koin module registration, and core-to-feature dependency rules.
2. WHEN global Compose UI rules are extracted THEN `guidelines/frontend/compose-ui.md` SHALL include Root/Content split, state/action/side-effect conventions, token usage, tap-handler rules, and screen composition rules.
3. WHEN global Supabase rules are extracted THEN `guidelines/backend/supabase.md` SHALL include Supabase client usage, service-role boundaries, schema verification before SQL, and mobile-client secret restrictions.
4. WHEN global Edge Function rules are extracted THEN `guidelines/backend/supabase-edge-functions.md` SHALL include request/response contract alignment, migration alignment, secret handling, and auth/JWT expectations.
5. WHEN Yawnly-specific facts are encountered THEN they SHALL remain in `projects/yawnly/` and SHALL NOT be added to global guidelines.

### Requirement 4 — Yawnly Project Context

**User Story:** As the project owner, I want a concise Yawnly project folder in the wiki, so that agents understand Yawnly without reading scattered repo docs first.

#### Acceptance Criteria

1. WHEN `projects/yawnly/context.md` is created THEN it SHALL include product summary, repo path, stack, modules, backend location, build commands, hard constraints, and what not to touch.
2. WHEN `projects/yawnly/linked-guidelines.md` is created THEN it SHALL list the exact global guidelines and golden-example categories Yawnly uses.
3. WHEN `projects/yawnly/decisions.md` is created THEN it SHALL contain at least 5 accepted project decisions from verified Yawnly docs.
4. WHEN `projects/yawnly/current-state.md` is created THEN it SHALL distinguish working, in-progress, blocked, and risky areas.
5. WHEN `projects/yawnly/repo-map.md` is created THEN it SHALL describe the actual Yawnly modules and call out stale or missing referenced docs.
6. WHEN `projects/yawnly/backlog.md` is created THEN it SHALL contain only `release-critical`, `next`, `later`, and `parked` sections.

### Requirement 5 — Yawnly Stale Reference Cleanup Plan

**User Story:** As the project owner, I want stale Yawnly documentation references identified before Hermes uses the project, so that the pilot does not start from false context.

#### Acceptance Criteria

1. WHEN Phase 1 is complete THEN `projects/yawnly/repo-map.md` SHALL list missing references to `plan/yawnly-cmp-implementation-plan.md`.
2. WHEN Phase 1 is complete THEN `projects/yawnly/repo-map.md` SHALL list missing or stale PRD references found in Yawnly docs.
3. WHEN Phase 1 is complete THEN `projects/yawnly/repo-map.md` SHALL state that `iosApp` is an Xcode host and not a Gradle module.
4. WHEN Phase 1 is complete THEN Yawnly stale references SHALL be either marked stale in the wiki or converted into a future task packet.

### Requirement 6 — Hermes Task Packet Pilot

**User Story:** As the project owner, I want one small Yawnly task packet, so that Hermes can prove the delegation loop before more projects are added.

#### Acceptance Criteria

1. WHEN the first task packet is created THEN it SHALL live under `projects/yawnly/task-packets/`.
2. WHEN the first task packet is created THEN it SHALL default to `mode: plan-only` unless explicitly approved otherwise.
3. WHEN the first task packet is created THEN it SHALL include end goal, output format, recommended approach, common failure handling, allowed paths, required context, non-goals, verification commands, stop conditions, and output requirements.
4. WHEN the first task packet is created THEN it SHALL avoid source-code implementation as its default action.
5. WHEN Hermes executes future implementation from a packet THEN it SHALL stop if a requested edit falls outside `Allowed Paths`.

### Requirement 6A — Hermes Provider and Curator Setup

**User Story:** As the project owner, I want Hermes model routing and Curator checks defined during setup, so that Hermes does the heavy lifting without creating unmanaged skill bloat.

#### Acceptance Criteria

1. WHEN Hermes is installed THEN `hermes doctor` SHALL pass before the Yawnly pilot begins.
2. WHEN the first model provider is configured THEN DeepSeek v4 Pro SHALL be tried first if Hermes exposes that model/provider.
3. WHEN DeepSeek v4 Pro is not available THEN the setup SHALL stop for human approval before choosing another DeepSeek model.
4. WHEN OpenAI is configured THEN it SHALL be added through a separate provider/fallback task packet after the Yawnly dry run, unless DeepSeek fails the initial dry run.
5. WHEN Hermes Curator exists in the installed version THEN `hermes curator status` SHALL be checked before real implementation work.
6. WHEN mission-critical custom skills exist THEN those skills SHALL be pinned before triggering Curator manually.
7. WHEN Curator owns skill lifecycle THEN MyWiki SHALL NOT duplicate Hermes skill usage tracking.

### Requirement 7 — Hermes and Agent Write Lanes

**User Story:** As the project owner, I want clear agent write lanes, so that Hermes and Claude can propose changes without silently rewriting canonical truth.

#### Acceptance Criteria

1. WHEN Hermes proposes a wiki change THEN the proposal SHALL go to `inbox/hermes-proposals/`.
2. WHEN Claude or Cursor proposes a wiki change THEN the proposal SHALL go to `inbox/claude-proposals/`.
3. WHEN a proposal is older than 14 days THEN it SHALL be promoted, deleted, or archived during weekly maintenance.
4. WHEN canonical guidelines or project decisions are changed THEN a human SHALL approve the change.

### Requirement 8 — Secrets Convention

**User Story:** As the project owner, I want a concrete secrets convention, so that no agent stores credentials in the wiki.

#### Acceptance Criteria

1. WHEN secrets are referenced THEN the wiki SHALL use environment variable names, Supabase secret names, `op://` references, or Bitwarden item names/IDs.
2. WHEN a raw API key, token, JWT, service role key, private key, or `.env` content is found in a wiki page THEN the page SHALL fail review.
3. WHEN sensitive billing or subscription details are added in a future phase THEN only the narrow sensitive file/folder SHALL be encrypted.

### Requirement 9 — Research Distillation

**User Story:** As the project owner, I want research to be distilled into reusable conclusions, so that only the useful 2-5 points survive.

#### Acceptance Criteria

1. WHEN research is captured THEN it SHALL start in `research/inbox/` or `inbox/quick-capture/`.
2. WHEN research is promoted THEN it SHALL include `Question`, `Keep`, `Decision`, `Rejected`, `Applies To`, `Next Action`, and `Sources`.
3. WHEN research has no decision, no reusable conclusion, and no next action THEN it SHALL remain non-canonical.

### Requirement 10 — Maintenance Without Early Tooling Bloat

**User Story:** As the project owner, I want lightweight maintenance rules, so that the wiki stays useful without becoming another project.

#### Acceptance Criteria

1. WHEN the wiki is launched THEN maintenance SHALL be manual.
2. WHEN weekly maintenance occurs THEN `inbox/quick-capture/`, `inbox/hermes-proposals/`, and `inbox/claude-proposals/` SHALL be reviewed.
3. WHEN quarterly maintenance is skipped twice THEN expansion SHALL pause until stale pages and inbox proposals are cleaned.
4. WHEN page size drift appears THEN a size-check script SHALL be created in a later task.
5. WHEN `rg` and `index.md` stop retrieving useful pages THEN qmd evaluation SHALL become a separate task packet.

---

## Non-Functional Requirements

### Documentation Architecture and Modularity

- Global facts live in `guidelines/`.
- Project-local facts live in `projects/<project>/`.
- Execution boundaries live in `task-packets/`.
- Research conclusions live in `research/accepted/`.
- Agent proposals live in `inbox/*-proposals/`.

### Security

- No raw secrets in the wiki.
- Hermes launch setup must restrict access to `MyWiki`, Yawnly, and a scratch directory.
- Day-job repositories must not be mounted or reachable by Hermes in the Yawnly pilot.

### Reliability

- Yawnly stale references must be identified before Hermes pilot execution.
- Task packets must define stop conditions before implementation modes are used.
- Canonical wiki updates require human approval.

### Usability

- `index.md` must be short enough for agents to read first.
- Project context must be readable without requiring recursive repo scans.
- Task packets must be reviewable from Telegram.

### Scope Control

- qmd, ops tracking, vendors, runbooks, postmortems, prompts, glossary, and global task boards are out of launch scope.
- More CMP projects are out of scope until the Yawnly loop succeeds.