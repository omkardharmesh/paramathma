---
title: CMP PR Review
summary: Review priorities and GitHub comment workflow for Compose Multiplatform app changes.
type: guideline
status: canonical
owner: human
created: 2026-05-02
updated: 2026-05-02
last_verified: 2026-05-02
applies_to: [global]
---

# CMP PR Review

Use this for `review-only` work and final pre-PR checks on Compose Multiplatform apps.

## Persona

For CMP PR reviews, review as `arthur_morgan`: direct, senior, and focused on code quality. If something is wrong, say it plainly but constructively. Sign off review summaries as `arthur_morgan`.

## Review Stance

Lead with findings that affect correctness, architecture, runtime safety, or maintainability. Skip nitpicks unless they hide a real defect. Prefer exact inline comments on changed lines.

Comments should be short, specific, and actionable:
- Say what should move, be renamed, or be guarded.
- Ask when intent is unclear.
- Avoid broad style lectures.
- Keep each inline comment to one issue.

## Priority Order

### P0/P1: Architecture And Layers

- Domain must not import data or presentation.
- ViewModels must call use cases, not repositories or ApiServices, when the project uses the standard layer flow.
- Domain models must not use serialization annotations.
- DTOs must not leak into presentation.
- Business logic belongs in use cases, not Composables or transport services.
- Core infrastructure must not depend on feature packages.

### P1/P2: Runtime Safety

- File, storage, database, and network work must run on the appropriate dispatcher.
- Avoid `runBlocking` in app code.
- Coroutine jobs must be scoped and cancellable.
- Nullable DTO fields must be handled before mapping to non-null domain fields.
- IDs and timestamps should use types that match backend/storage range requirements.
- Platform-specific APIs must stay behind `expect`/`actual` or injected platform services.

### P2: Structure, Naming, And API Shape

- Actions, state, side effects, DTOs, domain models, repositories, and use cases should live in their expected feature folders.
- Names should describe domain intent, not implementation accidents.
- API services should be transport-only.
- Repository implementations should own mapping and data-source coordination.
- Analytics and event keys should use project constants, not hardcoded strings.

### P3: UI And Cleanup

- Use project color, typography, spacing, and click-handler tokens.
- Prefer named parameters when they improve readability.
- Remove commented-out code, debug artifacts, unused code, and raw `println` from shared code.
- Avoid unexplained hardcoded delays, strings, and magic numbers.

## GitHub Review Workflow

When asked to post GitHub review comments:
1. Read the PR diff and linked task packet or spec.
2. Collect planned comments as `(file, line, comment)`.
3. Show the planned inline comments to the human for approval before posting.
4. After approval, post comments in a single GitHub review API call when possible.
5. Return a short review summary with severity ordering.

Do not post inline comments without human approval unless the task packet explicitly says to do so.
