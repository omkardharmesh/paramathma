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

## Done (Hermes Pilot)

- **Stale-doc reference map** (`task-packets/yawnly-20260502-stale-doc-reference-map.md`) — research-only completed 2026-05-02. Report: `inbox/hermes-proposals/archive/2026-05/yawnly-stale-doc-reference-map-20260502.md`.
- **Bridge-thinning** (`task-packets/yawnly-20260502-bridge-thinning.md`) — implemented and merged 2026-05-02. `Yawnly/AGENTS.md` and `Yawnly/CLAUDE.md` now point at MyWiki via a wiki-first First Read Protocol. 6 stale doc references removed across 6 files. PR: [#15](https://github.com/omkardharmesh/YawnlyCMP/pull/15).

## In Progress

- _To be filled by the human or by the next Hermes task packet._

## Blocked

- Hermes pilot is gated on remaining wiki skeleton completion and Hermes install. Stale-doc research and bridge-thinning are now resolved.

## Risks

- **Two network stacks coexist.** Supabase + legacy Ktor skeleton. New work picking the wrong stack is a recurring trap; D002 captures the rule.
- **Migration / function / DTO drift.** Any Edge Function or migration change must land in three places in lock-step (migration, function, Kotlin DTO). See `guidelines/backend/supabase-edge-functions.md`.
- **Server-authoritative entitlement.** D007 requires server-side IAP verification before persisting subscription state; client trust is a regression vector.

## Next Candidate Task Packets

1. Edge Function ↔ Kotlin DTO alignment audit. Future packet.
2. Story Library scaffolding spec. Future packet, depends on D008 still holding.
