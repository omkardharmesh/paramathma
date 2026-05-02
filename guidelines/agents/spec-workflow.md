---
title: Spec Workflow
summary: Strict file-driven requirements, design, tasks, and red-team workflow before substantial implementation.
type: guideline
status: canonical
owner: human
created: 2026-05-02
updated: 2026-05-02
last_verified: 2026-05-02
applies_to: [global]
---

# Spec Workflow

Use this for substantial `plan-only`, `implement-local`, and `implement-pr` work. It complements Hermes task packets; it does not replace `meta/task-packet-schema.md`.

## File Truth

For substantial implementation specs, truth lives in files, not chat memory:
- `requirements.md`
- `design.md`
- `tasks.md`

A Hermes task packet may request these files as the output of `plan-only` work, or may reference them as required context before implementation.

## Enforcement Contract

- Output only the required spec files unless the task packet asks for another deliverable.
- Risky claims must be labeled `Verified` with source and date, or `Out of Scope`.
- Unverified claims cannot become acceptance criteria.
- Ban fuzzy execution wording in acceptance criteria and implementation tasks: `optional`, `as needed`, `or equivalent`, `likely`, `confirm later`.
- Any discovered errata must update `requirements.md`, `design.md`, and `tasks.md` in the same turn if all three are affected.

## Requirements File

`requirements.md` should include:
- Feature goal, business context, and affected surfaces.
- Product alignment and user value.
- User stories.
- Acceptance criteria in `WHEN/IF ... THEN ... SHALL ...` form.
- Non-functional requirements for architecture, performance, security, reliability, and usability.

Acceptance criteria must be testable. They should not depend on guesses, future confirmation, or hidden chat context.

## Design File

`design.md` should include:
- Overview of the architecture change and primary flow.
- Alignment with project technical standards and structure.
- Existing files, functions, classes, and components to reuse.
- Integration points and contracts.
- Components with purpose, location, interface, and dependencies.
- Data models with fields, validation, and serialization requirements.
- Error scenarios with user impact, logging, and recovery behavior.
- Testing strategy with exact scope and pass criteria.

Runtime-risk specs must define invariants for serialization, state shape, startup defaults, concurrency, navigation, transactions, and rollback behavior as applicable.

## Tasks File

`tasks.md` should break implementation into ordered phases. Each task must include:
- Exact files to create or modify.
- Commands to run.
- Pass condition with expected output or assertion.
- Fail condition with the failure signal.
- Rollback note when runtime risk exists.
- Requirement references.
- A focused implementation prompt or handoff note.

No implementation starts until the red-team pass reports zero `P0/P1` spec defects.

## Red-Team Pass

Before implementation, run a hostile review of the spec for:
- API contract traps.
- Gradle, KMP target, dependency, and source-set traps.
- Serialization and runtime crash traps.
- State, lifecycle, navigation, threading, startup, and transaction invariants.
- Phasing drift where tasks are marked done but the feature is not shippable.

The red-team output should include only:
- `P0/P1` defects.
- Exact file and line references.
- Required patch list.

Patch the spec files before implementation begins.

## Relationship To Hermes Task Packets

Task packets still define mode, allowed paths, required context, stop conditions, and output requirements. Spec workflow files define the feature contract underneath a task packet.

If a task packet and spec conflict, stop and ask the human. Do not guess which source is newer.
