---
title: Yawnly Current State
summary: Live snapshot of what is working, in progress, blocked, and risky.
type: current-state
status: canonical
updated: 2026-05-02
---

# Yawnly — Current State

This file is short on purpose. Update when reality changes; do not let it grow.

## Working

- CMP shared module (`composeApp`) compiles for Android (`rtk ./gradlew :composeApp:compileKotlinAndroid`).
- Android app assembles (`rtk ./gradlew :androidApp:assembleDebug`).
- Supabase project is wired (`ynjiskzaxsqdovmugiwo`) with anon key access from the mobile client through `SupabaseBaseRepository`.
- Edge Functions are present for OTP login, IAP order/verify/restore, audio generation, story generation, and entitlement.
- Feature scaffolds for `onboarding`, `tonightsstory`, `storygenerator`, `storyreader`, `mythology`, `paywall`, `profile`, and `sharecard` exist under `composeApp/src/commonMain`.

## In Progress

- _To be filled by the human or by the first Hermes `research-only` pass._
- Candidates: stale-doc reference cleanup, Story Library scaffolding, Edge Function payload alignment with Kotlin DTOs.

## Blocked

- Hermes pilot is gated on completing this wiki skeleton and reviewing the Yawnly stale-doc map. Hermes install has not yet happened.

## Risks

- **Stale doc references.** `plan/yawnly-cmp-implementation-plan.md` is referenced from `Yawnly/AGENTS.md` and `Yawnly/CLAUDE.md` but does not exist. See `repo-map.md`.
- **Two network stacks coexist.** Supabase + legacy Ktor skeleton. New work picking the wrong stack is a recurring trap; D002 captures the rule.
- **Migration / function / DTO drift.** Any Edge Function or migration change must land in three places in lock-step (migration, function, Kotlin DTO). See `guidelines/backend/supabase-edge-functions.md`.
- **Server-authoritative entitlement.** D007 requires server-side IAP verification before persisting subscription state; client trust is a regression vector.

## Next Candidate Task Packets

1. `task-packets/yawnly-20260502-stale-doc-reference-map.md` — already created. `plan-only`/`research-only` first.
2. Yawnly bridge file thinning (point `Yawnly/AGENTS.md` and `Yawnly/CLAUDE.md` at the wiki, do not delete useful repo docs yet). Future packet, not yet created.
3. Edge Function ↔ Kotlin DTO alignment audit. Future packet.
4. Story Library scaffolding spec. Future packet, depends on D008 still holding.
