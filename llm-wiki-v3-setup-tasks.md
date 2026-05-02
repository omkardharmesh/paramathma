# Tasks Document — LLM Wiki v3 Setup + Yawnly Pilot

## Enforcement Contract

1. No implementation starts until a red-team review reports zero `P0/P1` defects across:
  - `llm-wiki-v3-setup-requirements.md`
  - `llm-wiki-v3-setup-design.md`
  - `llm-wiki-v3-setup-tasks.md`
2. This task plan is documentation/setup planning only. It does not authorize Yawnly source-code changes.
3. Each task defines exact files, commands, pass/fail output, and rollback notes.
4. Runtime-risk tasks require explicit human approval before execution.

---

## 2-Pass Contract

### Pass 1 — Authoring

Create only the three planning files:

- `/Users/eloelo/Downloads/MyWiki-v3-setup-requirements.md`
- `/Users/eloelo/Downloads/MyWiki-v3-setup-design.md`
- `/Users/eloelo/Downloads/MyWiki-v3-setup-tasks.md`

### Pass 2 — Red-Team

Review for:

- missing launch files
- path ambiguity
- Yawnly stale-reference handling gaps
- global/project boundary leaks
- task-packet safety gaps
- Hermes scope/sandbox gaps
- bloat creeping back into launch scope

Output format:

- `P0/P1` defects
- exact file/section references
- required patch list

Implementation begins only after zero `P0/P1`.

---

## Phase A — Spec Approval

### A1. Review three-file spec set

- File(s):
  - `/Users/eloelo/Downloads/MyWiki-v3-setup-requirements.md`
  - `/Users/eloelo/Downloads/MyWiki-v3-setup-design.md`
  - `/Users/eloelo/Downloads/MyWiki-v3-setup-tasks.md`
- Commands:
  - `rg "T[O]DO|T[B]D|confirm[ ]later|as[ ]needed|or[ ]equivalent|like[l]y|option[a]l" "/Users/eloelo/Downloads/MyWiki-v3-setup-requirements.md" "/Users/eloelo/Downloads/MyWiki-v3-setup-design.md" "/Users/eloelo/Downloads/MyWiki-v3-setup-tasks.md"`
- Pass condition:
  - No unresolved `P0/P1`.
  - No placeholder markers remain.
  - No implementation-critical fuzzy wording remains.
- Fail condition:
  - Any unresolved severity marker, placeholder marker, or implementation-critical fuzzy wording remains.
- Rollback:
  - Patch the spec files before implementation.
- Requirements:
  - R1-R10

### A2. Confirm launch root

- File(s):
  - `/Users/eloelo/Downloads/MyWiki-v3-setup-design.md`
  - `/Users/eloelo/Downloads/MyWiki-v3-setup-tasks.md`
- Commands:
  - `test ! -e "/Users/eloelo/Downloads/MyWiki"`
- Pass condition:
  - Command exits `0`, meaning the launch root does not exist.
- Fail condition:
  - Command exits non-zero, meaning the launch root exists.
- Rollback:
  - Do not delete anything. Produce a collision report and ask human how to proceed.
- Requirements:
  - R1

---

## Phase B — Create Minimal Wiki Skeleton

### B1. Create launch directories

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/`
  - `/Users/eloelo/Downloads/MyWiki/guidelines/frontend/`
  - `/Users/eloelo/Downloads/MyWiki/guidelines/backend/`
  - `/Users/eloelo/Downloads/MyWiki/guidelines/shared/`
  - `/Users/eloelo/Downloads/MyWiki/guidelines/agents/`
  - `/Users/eloelo/Downloads/MyWiki/golden-examples/cmp/`
  - `/Users/eloelo/Downloads/MyWiki/golden-examples/supabase/`
  - `/Users/eloelo/Downloads/MyWiki/golden-examples/fastapi/`
  - `/Users/eloelo/Downloads/MyWiki/golden-examples/spring/`
  - `/Users/eloelo/Downloads/MyWiki/projects/_template/`
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/`
  - `/Users/eloelo/Downloads/MyWiki/research/inbox/`
  - `/Users/eloelo/Downloads/MyWiki/research/accepted/`
  - `/Users/eloelo/Downloads/MyWiki/research/rejected/`
  - `/Users/eloelo/Downloads/MyWiki/inbox/quick-capture/`
  - `/Users/eloelo/Downloads/MyWiki/inbox/hermes-proposals/`
  - `/Users/eloelo/Downloads/MyWiki/inbox/claude-proposals/`
  - `/Users/eloelo/Downloads/MyWiki/meta/`
- Commands:
  - `test -d "/Users/eloelo/Downloads/MyWiki/guidelines/frontend"`
  - `test -d "/Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets"`
  - `test -d "/Users/eloelo/Downloads/MyWiki/inbox/hermes-proposals"`
- Pass condition:
  - All commands exit `0`.
- Fail condition:
  - Any command exits non-zero.
- Rollback:
  - If no useful files exist inside the new root, remove only `/Users/eloelo/Downloads/MyWiki` after human approval.
- Requirements:
  - R1

### B2. Create root entry files

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/index.md`
  - `/Users/eloelo/Downloads/MyWiki/README.md`
  - `/Users/eloelo/Downloads/MyWiki/AGENTS.md`
  - `/Users/eloelo/Downloads/MyWiki/HERMES.md`
- Commands:
  - `test -f "/Users/eloelo/Downloads/MyWiki/index.md"`
  - `test -f "/Users/eloelo/Downloads/MyWiki/AGENTS.md"`
  - `test -f "/Users/eloelo/Downloads/MyWiki/HERMES.md"`
  - `rg "First Read Protocol|projects/yawnly/context.md|meta/wikiignore.md" "/Users/eloelo/Downloads/MyWiki/index.md"`
- Pass condition:
  - All files exist.
  - `index.md` contains first-read protocol, Yawnly context link, and wikiignore link.
- Fail condition:
  - Any file missing or required link missing.
- Rollback:
  - Restore prior file contents from git if wiki repo exists; otherwise patch the files.
- Requirements:
  - R1, R2

### B3. Create meta schema files

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/meta/page-schema.md`
  - `/Users/eloelo/Downloads/MyWiki/meta/task-packet-schema.md`
  - `/Users/eloelo/Downloads/MyWiki/meta/research-note-schema.md`
  - `/Users/eloelo/Downloads/MyWiki/meta/maintenance-checklist.md`
  - `/Users/eloelo/Downloads/MyWiki/meta/wikiignore.md`
- Commands:
  - `rg "title:|summary:|last_verified:|applies_to:" "/Users/eloelo/Downloads/MyWiki/meta/page-schema.md"`
  - `rg "Allowed Paths|Stop Conditions|Output Required" "/Users/eloelo/Downloads/MyWiki/meta/task-packet-schema.md"`
  - `rg "Question|Keep|Decision|Rejected|Next Action|Sources" "/Users/eloelo/Downloads/MyWiki/meta/research-note-schema.md"`
  - `rg "Agents must not bulk-load" "/Users/eloelo/Downloads/MyWiki/meta/wikiignore.md"`
- Pass condition:
  - All required schema markers are present.
- Fail condition:
  - Any required schema marker is missing.
- Rollback:
  - Patch missing schema sections before proceeding.
- Requirements:
  - R2, R7, R9, R10

---

## Phase C — Extract First Global Guidelines

### C1. Create CMP architecture guideline

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/guidelines/frontend/compose-multiplatform.md`
- Source files:
  - `/Users/eloelo/Downloads/Yawnly/AGENTS.md`
  - `/Users/eloelo/Downloads/Yawnly/docs/agent-architecture.md`
  - `/Users/eloelo/Downloads/Yawnly/CLAUDE.md`
- Commands:
  - `rg "ViewModel|UseCase|Repository|RepositoryImpl|ApiService|@Serializable|Koin|core" "/Users/eloelo/Downloads/MyWiki/guidelines/frontend/compose-multiplatform.md"`
- Pass condition:
  - Guideline includes layer flow, DTO/domain rules, Koin rules, and core-to-feature dependency rule.
- Fail condition:
  - Any required CMP rule is missing.
- Rollback:
  - Patch the guideline; do not copy entire Yawnly docs.
- Requirements:
  - R3

### C2. Create Compose UI guideline

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/guidelines/frontend/compose-ui.md`
- Source files:
  - `/Users/eloelo/Downloads/Yawnly/DESIGN.md`
  - `/Users/eloelo/Downloads/Yawnly/docs/yawnly-ui-ux-polish-guide.md`
  - `/Users/eloelo/Downloads/Yawnly/AGENTS.md`
- Commands:
  - `rg "Root|Content|state|onAction|side effect|token|tap|click" "/Users/eloelo/Downloads/MyWiki/guidelines/frontend/compose-ui.md"`
- Pass condition:
  - Guideline includes Root/Content split, state/action/effect rules, token rules, tap-handler rule.
- Fail condition:
  - Any required UI rule is missing.
- Rollback:
  - Patch guideline and move Yawnly brand-specific rules to Yawnly project context.
- Requirements:
  - R3

### C3. Create Supabase guideline

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/guidelines/backend/supabase.md`
- Source files:
  - `/Users/eloelo/Downloads/Yawnly/AGENTS.md`
  - `/Users/eloelo/Downloads/Yawnly/docs/agent-architecture.md`
  - `/Users/eloelo/Downloads/Yawnly/plan/v2/komodo/komodo_supabase_wiring_map.md`
- Commands:
  - `rg "anon|service role|RLS|schema|SQL|secret|DTO|migration" "/Users/eloelo/Downloads/MyWiki/guidelines/backend/supabase.md"`
- Pass condition:
  - Guideline includes secret boundary, RLS/schema checks, migration/DTO/function alignment.
- Fail condition:
  - Any required Supabase rule is missing.
- Rollback:
  - Patch guideline and keep Yawnly-specific SKU/plan details out.
- Requirements:
  - R3, R8

### C4. Create Supabase Edge Functions guideline

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/guidelines/backend/supabase-edge-functions.md`
- Source files:
  - `/Users/eloelo/Downloads/Yawnly/plan/otp_integration.md`
  - `/Users/eloelo/Downloads/Yawnly/plan/v2/komodo/supabase_iap_server_impl_spec.md`
  - `/Users/eloelo/Downloads/Yawnly/plan/v2/komodo/komodo_supabase_wiring_map.md`
- Commands:
  - `rg "env|JWT|request|response|webhook|idempot|migration|error" "/Users/eloelo/Downloads/MyWiki/guidelines/backend/supabase-edge-functions.md"`
- Pass condition:
  - Guideline includes env secret handling, auth/JWT expectations, response contract, migration alignment, and webhook caution.
- Fail condition:
  - Any required Edge Function rule is missing.
- Rollback:
  - Patch guideline and keep provider-specific details as examples, not universal rules.
- Requirements:
  - R3, R8

### C5. Create Hermes delegation guideline

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/guidelines/agents/hermes-delegation.md`
- Source files:
  - `/Users/eloelo/Downloads/MyWiki/llm-wiki-plan-v3.md`
  - Hermes Atlas guide
- Commands:
  - `rg "plan-only|implement-local|Allowed Paths|branch|proposal|Telegram|Hermes" "/Users/eloelo/Downloads/MyWiki/guidelines/agents/hermes-delegation.md"`
- Pass condition:
  - Guideline includes delegation modes, branch rule, allowed paths, proposal lanes, and Telegram default behavior.
- Fail condition:
  - Any required delegation rule is missing.
- Rollback:
  - Patch guideline before Hermes setup.
- Requirements:
  - R6, R7

---

## Phase D — Create Yawnly Project Wiki

### D1. Create Yawnly context

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/context.md`
- Source files:
  - `/Users/eloelo/Downloads/Yawnly/CLAUDE.md`
  - `/Users/eloelo/Downloads/Yawnly/AGENTS.md`
  - `/Users/eloelo/Downloads/Yawnly/docs/agent-architecture.md`
- Commands:
  - `rg "bedtime story|/Users/eloelo/Downloads/Yawnly|composeApp|androidApp|iosApp|supabase|BaseRepository|Room" "/Users/eloelo/Downloads/MyWiki/projects/yawnly/context.md"`
- Pass condition:
  - Context includes product summary, repo path, stack, modules, backend, build commands, and hard constraints.
- Fail condition:
  - Any required project context item is missing.
- Rollback:
  - Patch context before creating task packets.
- Requirements:
  - R4

### D2. Create Yawnly linked guidelines

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/linked-guidelines.md`
- Commands:
  - `rg "compose-multiplatform.md|compose-ui.md|supabase.md|supabase-edge-functions.md|hermes-delegation.md" "/Users/eloelo/Downloads/MyWiki/projects/yawnly/linked-guidelines.md"`
- Pass condition:
  - All required guideline links are present.
- Fail condition:
  - Any required guideline link is missing.
- Rollback:
  - Patch linked guidelines.
- Requirements:
  - R4

### D3. Create Yawnly decisions

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/decisions.md`
- Source files:
  - `/Users/eloelo/Downloads/Yawnly/plan/v2/pending/ideation.md`
  - `/Users/eloelo/Downloads/Yawnly/AGENTS.md`
  - `/Users/eloelo/Downloads/Yawnly/docs/agent-architecture.md`
  - `/Users/eloelo/Downloads/Yawnly/plan/v2/pending/release-pending.md`
- Commands:
  - `rg "D001|D002|D003|D004|D005|Supabase|audio-only|RAG|PII|subscription" "/Users/eloelo/Downloads/MyWiki/projects/yawnly/decisions.md"`
- Pass condition:
  - At least 5 decisions are present.
  - Supabase, no Room for product data, audio before buddy, no RAG for story generation, PII placeholders, and server-authoritative subscription truth are represented.
- Fail condition:
  - Fewer than 5 decisions or any required decision absent.
- Rollback:
  - Patch decisions; keep project-only decisions out of global `guidelines/`.
- Requirements:
  - R4

### D4. Create Yawnly current state and repo map

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/current-state.md`
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/repo-map.md`
- Commands:
  - `rg "Working|In Progress|Blocked|Risks|Next Candidate Task Packets" "/Users/eloelo/Downloads/MyWiki/projects/yawnly/current-state.md"`
  - `rg "composeApp|androidApp|iosApp|supabase|missing|stale|plan/yawnly-cmp-implementation-plan|yawnly-prd" "/Users/eloelo/Downloads/MyWiki/projects/yawnly/repo-map.md"`
- Pass condition:
  - Current-state sections exist.
  - Repo map includes actual modules and stale reference callouts.
- Fail condition:
  - Required sections or stale reference callouts missing.
- Rollback:
  - Patch both files before Hermes setup.
- Requirements:
  - R4, R5

### D5. Create Yawnly backlog

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/backlog.md`
- Commands:
  - `rg "release-critical|next|later|parked" "/Users/eloelo/Downloads/MyWiki/projects/yawnly/backlog.md"`
- Pass condition:
  - All four allowed backlog sections exist.
- Fail condition:
  - Any required section missing or backlog exceeds 150 lines.
- Rollback:
  - Patch file or move oversized items into task packets.
- Requirements:
  - R4

---

## Phase E — Create First Task Packet

### E1. Create task packet README

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/README.md`
- Commands:
  - `rg "status|mode|Allowed Paths|Stop Conditions|review" "/Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/README.md"`
- Pass condition:
  - README explains status, mode, allowed paths, stop conditions, and review rules.
- Fail condition:
  - Any required concept missing.
- Rollback:
  - Patch README.
- Requirements:
  - R6

### E2. Create no-code stale-doc task packet

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/yawnly-20260502-stale-doc-reference-map.md`
- Commands:
  - `rg "mode: plan-only|Allowed Paths|Required Context|Stop Conditions|Output Required|plan/yawnly-cmp-implementation-plan|yawnly-prd" "/Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/yawnly-20260502-stale-doc-reference-map.md"`
- Pass condition:
  - Packet defaults to `plan-only`.
  - Packet includes stale-doc focus, allowed paths, required context, stop conditions, and output required.
- Fail condition:
  - Packet permits source-code edits by default or omits required sections.
- Rollback:
  - Patch packet before Hermes uses it.
- Requirements:
  - R5, R6

---

## Phase F — Hermes Setup Plan

### F1. Prepare Hermes setup checklist

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/hermes-installation-plan.md`
  - `/Users/eloelo/Downloads/MyWiki/guidelines/agents/hermes-delegation.md`
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/yawnly-20260502-stale-doc-reference-map.md`
- Commands:
  - `test -f "/Users/eloelo/Downloads/MyWiki/hermes-installation-plan.md"`
  - `rg "DeepSeek v4 Pro|OpenAI|hermes doctor|hermes model|hermes gateway setup|hermes curator status|numeric Telegram user ID" "/Users/eloelo/Downloads/MyWiki/hermes-installation-plan.md"`
  - `rg "Docker|separate macOS user|Telegram allowlist|numeric Telegram user ID|approval policies|Yawnly only|Curator|DeepSeek first|OpenAI" "/Users/eloelo/Downloads/MyWiki/guidelines/agents/hermes-delegation.md"`
- Pass condition:
  - Hermes installation plan exists and names DeepSeek first, OpenAI later, and Curator status before real work.
  - Hermes checklist includes restricted access, approval policies, numeric Telegram user ID, Yawnly-only pilot, model routing, and Curator checkpoint.
- Fail condition:
  - Hermes installation plan is missing.
  - Hermes checklist allows unrestricted local access.
- Rollback:
  - Patch delegation guideline before installing or running Hermes.
- Requirements:
  - R6, R7, R8

### F1A. Add Curator safety checkpoint

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/hermes-installation-plan.md`
  - `/Users/eloelo/Downloads/MyWiki/guidelines/agents/hermes-delegation.md`
- Commands:
  - `rg "hermes curator status|pin any mission-critical custom skill|MyWiki does not track Hermes skill usage|Curator owns skill cleanup" "/Users/eloelo/Downloads/MyWiki/hermes-installation-plan.md"`
- Pass condition:
  - Install plan requires Curator status check.
  - Install plan states that MyWiki does not duplicate Hermes skill lifecycle tracking.
  - Install plan requires pinning mission-critical custom skills before manual Curator runs.
- Fail condition:
  - Curator can run without a pin/status checkpoint.
- Rollback:
  - Patch Hermes install plan before running Hermes implementation tasks.
- Requirements:
  - R6A, R7, R10

### F2. Define first Hermes dry run

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/yawnly-20260502-stale-doc-reference-map.md`
- Commands:
  - `rg "research-only|plan-only|implement-local|review-only" "/Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/yawnly-20260502-stale-doc-reference-map.md"`
- Pass condition:
  - Packet or README states the dry-run order:
    1. `research-only`
    2. `plan-only`
    3. human review
    4. later `implement-local` only after approval
- Fail condition:
  - Dry run jumps directly to implementation.
- Rollback:
  - Patch task packet.
- Requirements:
  - R6, R7

---

## Phase G — Validation and Red-Team Before First Real Use

### G1. Validate no launch-scope bloat

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/`
- Commands:
  - `test ! -d "/Users/eloelo/Downloads/MyWiki/ops"`
  - `test ! -d "/Users/eloelo/Downloads/MyWiki/prompts"`
  - `test ! -f "/Users/eloelo/Downloads/MyWiki/TASKS.md"`
  - `test ! -d "/Users/eloelo/Downloads/MyWiki/runbooks"`
- Pass condition:
  - All commands exit `0`.
- Fail condition:
  - Any deferred folder/file exists at launch.
- Rollback:
  - Remove launch-bloat folder/file after confirming no unique canonical content exists; otherwise move content into the correct existing folder.
- Requirements:
  - R10

### G2. Validate no raw secrets

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/`
- Commands:
  - `rg "sk-|service_role|eyJ|-----BEGIN|SUPABASE_SERVICE_ROLE_KEY=.*|OPENAI_API_KEY=.*|ANTHROPIC_API_KEY=.*" "/Users/eloelo/Downloads/MyWiki"`
- Pass condition:
  - No raw secret-looking values are found.
- Fail condition:
  - Any raw secret-looking value is found.
- Rollback:
  - Remove the secret immediately, rotate it if it was real, and replace with a reference such as env var name or `op://` reference.
- Requirements:
  - R8

### G3. Red-team the wiki setup

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/`
- Commands:
  - `rg "allowed paths|stop conditions|human approval|plan-only|Yawnly only|do not recursively" "/Users/eloelo/Downloads/MyWiki"`
- Pass condition:
  - Required safety phrases are present in root agent docs, Hermes guideline, and task packet schema.
- Fail condition:
  - Safety phrases are missing from any of the three control surfaces.
- Rollback:
  - Patch missing safety surfaces before Hermes setup.
- Requirements:
  - R6, R7, R10

---

## Phase H — Future First Workflow

### H1. Execute Yawnly stale-doc task through Hermes

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/yawnly-20260502-stale-doc-reference-map.md`
  - `/Users/eloelo/Downloads/MyWiki/inbox/hermes-proposals/`
- Commands:
  - Hermes Telegram command, exact text:
    - `Yawnly: run research-only on task packet projects/yawnly/task-packets/yawnly-20260502-stale-doc-reference-map.md. Do not edit source code. Return report only.`
- Pass condition:
  - Hermes returns a report.
  - Hermes does not edit Yawnly source code.
  - Hermes proposes wiki updates only in `inbox/hermes-proposals/`.
- Fail condition:
  - Hermes edits files outside approved paths or attempts source-code implementation.
- Rollback:
  - Stop Hermes task.
  - Revert unapproved file changes.
  - Patch task packet stop conditions.
- Requirements:
  - R5, R6, R7

### H2. Decide whether to proceed to implementation loop

- File(s):
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/current-state.md`
  - `/Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/`
- Commands:
  - Human review only.
- Pass condition:
  - Human confirms Hermes report reduced review effort and obeyed paths.
- Fail condition:
  - Human reports excessive guidance, bad context use, or path violation.
- Rollback:
  - Keep system in `research-only`/`plan-only` mode and patch wiki/task packet instructions.
- Requirements:
  - R6, R7, R10

---

## Expansion Gate

Do not add other CMP projects until all are true:

1. Yawnly context and linked guidelines are complete.
2. Stale Yawnly references are mapped.
3. One Hermes dry run succeeds from Telegram.
4. Hermes obeys the task packet.
5. Human confirms the workflow saved time.

After this gate, create a new spec for the second CMP project rather than bulk-migrating all projects.