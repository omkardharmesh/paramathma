# Hermes Installation Plan — MyWiki + Yawnly Pilot

**Status:** Draft for review  
**Date:** 2026-05-02  
**Scope:** Plan only. Do not install or run Hermes until the wiki skeleton and Yawnly context are ready.

---

## 1. Goal

Set up Hermes as the execution/control layer for `MyWiki` and the Yawnly pilot.

Initial model provider:

- DeepSeek v4 Pro, if Hermes exposes that exact model/provider during setup.

Later provider:

- OpenAI, added after the Yawnly pilot loop works.

Important constraint:

> Hermes should not start as an unrestricted local agent. It must first be connected only to `MyWiki`, Yawnly, and a scratch directory.

---

## 2. Verified Inputs

Verified from Hermes Atlas guide, retrieved 2026-05-02:

- Hermes installs via:

```shell
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | sh
```

- First run:

```shell
hermes
```

- Health check:

```shell
hermes doctor
```

- Telegram gateway setup is described as:

```shell
hermes gateway setup
```

- Hermes supports model providers and custom/API model routing.
- Hermes has memory, skills, sessions, cron, Telegram/gateway control, profiles, and approval policies.
- Hermes can use Docker/backend isolation and approval policies for safer execution.

Unverified until install:

- exact Hermes model/provider name for "DeepSeek v4 Pro"
- exact OpenAI model selection command in the installed Hermes version
- exact approval-policy command names in the installed Hermes version
- exact Curator command behavior in the installed Hermes version

Rule:

> During installation, use `hermes doctor`, `hermes model`, `hermes --help`, `hermes gateway --help`, and `hermes curator --help` to verify exact commands before saving them into MyWiki.

---

## 3. Target Local Layout

```text
/Users/eloelo/Downloads/MyWiki/        # wiki root and planning docs
/Users/eloelo/Downloads/Yawnly/        # first pilot repo
/Users/eloelo/Downloads/hermes-scratch/ # temporary reports/output
```

Hermes should not receive access to other Downloads projects during the pilot.

---

## 4. Install Sequence

### Step 1 — Preflight

Check:

- `MyWiki` exists.
- Yawnly exists.
- `hermes-scratch` exists or can be created.
- No day-job repo path is mounted into Hermes.
- DeepSeek API key is available through a secret manager or environment variable, not written into MyWiki.
- OpenAI key is not needed for the first run.

Commands to verify later:

```shell
test -d "/Users/eloelo/Downloads/MyWiki"
test -d "/Users/eloelo/Downloads/Yawnly"
mkdir -p "/Users/eloelo/Downloads/hermes-scratch"
```

### Step 2 — Install Hermes

Command from Hermes Atlas:

```shell
curl -fsSL https://hermes-agent.nousresearch.com/install.sh | sh
```

Then:

```shell
hermes doctor
```

Pass condition:

- `hermes doctor` reports a healthy install or gives only fixable configuration warnings.

Stop condition:

- If install modifies shell config unexpectedly, stop and review before continuing.

### Step 3 — Configure DeepSeek v4 Pro First

Open Hermes model configuration:

```shell
hermes model
```

Expected action:

- Choose DeepSeek or OpenAI-compatible custom provider if DeepSeek v4 Pro is exposed that way.
- Store the API key in Hermes/provider config, not in MyWiki.
- Verify the exact configured model name and record it later in `MyWiki/HERMES.md`.

Stop condition:

- If the exact "DeepSeek v4 Pro" model is not available, do not guess a replacement. Choose the closest DeepSeek model only after human approval.

### Step 4 — Keep OpenAI As Later Provider

Do not configure OpenAI during the first Yawnly dry run unless DeepSeek fails tool-calling or planning quality.

Later OpenAI integration task:

```text
projects/yawnly/task-packets/yawnly-openai-provider-fallback.md
```

That packet should define:

- OpenAI key source
- model choice
- routing rule
- when to use OpenAI instead of DeepSeek
- cost guardrail

### Step 5 — Check Hermes Curator Before Real Work

Hermes Curator is relevant because Hermes can create skills as it learns. Curator handles skill lifecycle so `MyWiki` does not need to track Hermes skills.

Commands to verify after install:

```shell
hermes curator status
hermes curator --help
```

Expected action:

- Check whether Curator exists in the installed Hermes version.
- Check whether Curator is enabled.
- Check active/stale/archived counts if available.
- Check the pinned skill list if available.
- Pin any mission-critical custom skill before enabling/updating Curator.

Important rule:

> MyWiki does not track Hermes skill usage. Hermes Curator owns skill cleanup. MyWiki only records stable project truth and task packet contracts.

Stop condition:

- If Curator has no dry-run/preview mode in the installed version, do not trigger a manual Curator run until critical custom skills are pinned or the skills folder is confirmed disposable.

Curator policy for pilot:

- It is acceptable to leave Curator enabled for default cleanup if no custom mission-critical skills exist yet.
- Do not create MyWiki pages that duplicate Curator state.
- Do not treat archived Hermes skills as deleted; archived skills are recoverable through Hermes.

### Step 6 — Configure Telegram Gateway

Command from Hermes Atlas:

```shell
hermes gateway setup
```

Telegram safety rules:

- use numeric Telegram user ID, not username
- allowlist only your account
- do not expose group chat access during pilot
- do not enable multi-user/shared profile during pilot

Pass condition:

- You can send a Telegram message and receive a response.

First test message:

```text
Read /Users/eloelo/Downloads/MyWiki/index.md and summarize the first-read protocol. Do not edit files.
```

### Step 7 — Configure Workspace Access

Pilot access should be:

```text
read/write: /Users/eloelo/Downloads/MyWiki
read/write: /Users/eloelo/Downloads/Yawnly
read/write: /Users/eloelo/Downloads/hermes-scratch
deny: other project directories
deny: day-job repos
deny: secrets folders unless explicitly needed
```

Preferred:

- Docker backend or separate macOS user.

Minimum acceptable pilot:

- launch Hermes from a context where only `MyWiki`, Yawnly, and scratch are intentionally used
- task packets enforce allowed paths
- approval policies enabled for destructive operations

### Step 8 — First Dry Run

Do not ask Hermes to implement code first.

First dry run:

```text
Yawnly: run research-only on /Users/eloelo/Downloads/MyWiki/projects/yawnly/task-packets/yawnly-20260502-stale-doc-reference-map.md.
Do not edit source code.
Return report only.
```

Pass condition:

- Hermes reads MyWiki context.
- Hermes reads Yawnly docs only as required by task packet.
- Hermes does not edit Yawnly source code.
- Hermes returns a useful report.

Fail condition:

- Hermes edits outside allowed paths.
- Hermes ignores task packet.
- Hermes tries to implement before approval.
- Hermes cannot use DeepSeek reliably for tool-calling.

Fallback:

- Patch task packet/HERMES.md if instruction issue.
- Switch model/provider only if model quality is the failure.

---

## 5. Provider Strategy

### DeepSeek v4 Pro — First Provider

Use for:

- research-only tasks
- plan-only tasks
- repo-map summaries
- stale-doc analysis
- draft task packets
- low-risk documentation work
- detached long-running tasks where the task packet is explicit

Watch for:

- tool-calling reliability
- ability to obey allowed paths
- ability to stop instead of guessing
- quality of architecture reasoning on Yawnly

### OpenAI — Later Provider

Use after first loop works, for:

- high-risk implementation tasks
- debugging failing builds
- complex cross-file refactors
- final review before PR
- tasks where DeepSeek fails tool-calling, stop-condition discipline, or architecture reasoning

OpenAI integration should be a separate task packet, not part of the initial install.

Routing rule:

```text
DeepSeek first for research-only and plan-only.
OpenAI/frontier model only when the task is implementation-risky, debugging-heavy, or DeepSeek fails the dry run.
```

---

## 6. MyWiki Updates After Install

After Hermes install succeeds, update:

```text
HERMES.md
guidelines/agents/hermes-delegation.md
projects/yawnly/current-state.md
projects/yawnly/task-packets/README.md
```

Record:

- installed Hermes version
- exact DeepSeek model/provider name
- whether Telegram gateway works
- whether approval policies are enabled
- where scratch output goes
- what directories Hermes can access

Do not record:

- API keys
- Telegram bot token
- OpenAI key
- raw provider secrets

---

## 7. Approval Gate

Hermes is considered ready for the Yawnly pilot only after:

1. `hermes doctor` passes.
2. DeepSeek provider returns a basic answer.
3. Telegram gateway responds to your numeric allowlisted user.
4. Hermes can read `MyWiki/index.md`.
5. Hermes can run the first `research-only` task without edits.
6. No source code changes occur during the dry run.

Only after this should `implement-local` be tested.