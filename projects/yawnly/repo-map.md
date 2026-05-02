---
title: Yawnly Repo Map
summary: Module layout, feature packages, Supabase backend, key docs, and stale references.
type: repo-map
status: canonical
updated: 2026-05-02
---

# Yawnly — Repo Map

Repo path: `/Users/eloelo/Downloads/Yawnly`

## Actual Modules

- `composeApp/` — KMP **library**. Almost all Yawnly Kotlin lives here under `src/commonMain`, with `src/androidMain` and `src/iosMain` for platform code.
- `androidApp/` — Android **application** module. Manifest, `Application`, `MainActivity`, `AndroidAppModule`. Depends on `composeApp`.
- `iosApp/` — Xcode host that embeds the Compose framework. **Not a Gradle module.** Verified 2026-05-02: not registered in `settings.gradle.kts`, no `iosApp/build.gradle.kts`.
- `supabase/` — backend: SQL migrations + Edge Functions (Deno).

Other top-level project folders (e.g. `commonMain`, `myserver`, `modal`, `datasets`, `komodo`, `skills`, `graphify-out`, `plan`, `docs`) exist; treat them as auxiliary unless a task packet explicitly lists them.

## Source Tree (composeApp)

```
composeApp/src/commonMain/kotlin/com/techmo/yawnly/
  feature/<name>/
    data/dto/           # @Serializable + @SerialName + toDomain()
    data/network/       # Supabase or Ktor ApiService
    data/repository/    # *RepositoryImpl extends SupabaseBaseRepository
    domain/model/       # plain data classes
    domain/repository/  # interfaces
    domain/usecase/     # interface + Impl
    presentation/action/, state/, sideEffects/, ui/, viewmodel/
    di/<Feature>Module.kt
    presentation/navigation/<Feature>Navigation.kt
  core/
    domain/             # BaseRepository, SupabaseBaseRepository, shared domain
    presentation/       # AppViewModel, BaseViewModel, shared composables
    feature/auth/, datastore/, navigation/, tts/, analytics/, home/, ktor/, permissions/, db/, ...
  app/
    Route.kt, App.kt, KoinInit.android.kt (in androidMain), KoinInit.ios.kt (in iosMain)
```

## Feature Packages

- `onboarding`
- `tonightsstory`
- `storygenerator`
- `storyreader`
- `mythology`
- `paywall`
- `profile`
- `sharecard`

## Core Packages

`core/feature/auth`, `core/feature/datastore`, `core/feature/navigation`, `core/feature/tts`, `core/feature/analytics`, `core/feature/home`, `core/feature/ktor`, `core/feature/permissions`, `core/feature/db`, plus shared base classes in `core/domain` and `core/presentation`.

`core/` must not import `feature/*` types.

## Supabase Backend

- Project: `ynjiskzaxsqdovmugiwo`
- URL: `https://ynjiskzaxsqdovmugiwo.supabase.co`
- Migrations: `supabase/migrations/` (verified 2026-05-02: 4 files present).
- Edge Functions present (verified 2026-05-02 via `ls supabase/functions/`):
  - `_shared`
  - `apple-webhook`
  - `generate-audio`
  - `generate-audio-chirp`
  - `generate-story`
  - `get-entitlement`
  - `google-webhook`
  - `payments-create-order`
  - `payments-restore-purchase`
  - `payments-verify-order`
  - `truecaller-login`
  - `tts-route`
- A `graphify-out/` directory may exist inside `supabase/`; treat as a generated artifact, not canonical content.

## Important Docs (live, in-repo)

| Doc                                       | Use it for                                                               |
| ----------------------------------------- | ------------------------------------------------------------------------ |
| `Yawnly/AGENTS.md`                        | Build commands, architecture gotchas, review gates, UI conventions       |
| `Yawnly/CLAUDE.md`                        | Product summary, do-not-touch list, Supabase project details             |
| `Yawnly/docs/agent-architecture.md`       | Primary clean-architecture reference (layers, DI, anti-patterns)         |
| `Yawnly/docs/yawnly-ui-ux-polish-guide.md`| UI tokens, animations, spacing                                           |
| `Yawnly/docs/codebase-review.md`          | Optional dated full-repo snapshot for audits                             |
| `Yawnly/DESIGN.md`                        | Visual / product design notes                                            |
| `Yawnly/plan/v2/pending/ideation.md`      | Product ideation notes                                                   |
| `Yawnly/plan/v2/pending/release-pending.md` | Release queue and pending work                                         |
| `Yawnly/plan/otp_integration.md`          | OTP login Edge Function design                                           |
| `Yawnly/plan/v2/komodo/komodo_supabase_wiring_map.md` | Supabase wiring map for IAP/payments                         |
| `Yawnly/plan/v2/komodo/supabase_iap_server_impl_spec.md` | Server-side IAP verification spec                         |

## Missing / Stale References

Verified 2026-05-02:

- `plan/yawnly-cmp-implementation-plan.md` is referenced by `Yawnly/CLAUDE.md`, `Yawnly/AGENTS.md`, `Yawnly/docs/agent-architecture.md`, and `Yawnly/plan/v2/pending/release-pending.md`, but the file **does not exist**. Treat as stale; do not trust references to it. (Verified by Hermes dry run, 2026-05-02; full map in `inbox/hermes-proposals/yawnly-stale-doc-reference-map-20260502.md`.)
- `plan/yawnly-prd-v4.md` is referenced by `release-pending.md` but does not exist.
- `plan/yawnly-prd-v3.md` is referenced by `docs/agent-architecture.md` but does not exist (the citing line itself notes v4 is current).
- `plan/release-v1.1-findings.md` is referenced by `release-pending.md` but does not exist.
- `komodo-app-integration-detailed.md` is referenced by `komodo_integration_impl_spec.md` (cross-project, hold).
- `iosApp` is an Xcode host, **not** a Gradle module. References that imply iOS-side Gradle commands are stale.
- `docs/llm-agent-feature-implementation-guide.md` was removed; its content moved into `docs/agent-architecture.md`. Older notes referring to the removed path are stale.

## Generated Reports To Avoid By Default

- `graphify-out/` (any module).
- `*ScreenSnapshots*`, `build/`, `.gradle/`, `.kotlin/`, `.idea/`, `.obsidian/`.
- Per-module knowledge-graph reports under `composeApp/graphify-out/` and `iosApp/graphify-out/`.

These are not canonical context. Read only when a task packet explicitly lists them.

## Future Cleanup (Not At Launch)

- Decide whether `plan/yawnly-cmp-implementation-plan.md` should be re-created or its references removed from `AGENTS.md` and `CLAUDE.md`. Tracked as the first Yawnly task packet.
- Thin `Yawnly/AGENTS.md` and `Yawnly/CLAUDE.md` over time so that they point at the wiki for global rules and keep only Yawnly-specific deltas.
