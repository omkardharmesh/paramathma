---
title: Yawnly Project Context
summary: CMP bedtime story app. Supabase backend. First Hermes pilot.
type: project-context
status: canonical
updated: 2026-05-03
---

# Yawnly — Project Context

## Product

- Yawnly is a Compose Multiplatform (CMP) bedtime story app for Indian kids aged 1–10+.
- Generates personalized AI stories and reads them aloud via Edge TTS.
- Monetizes via weekly/monthly subscriptions.

## Repo

- Path: `/Users/eloelo/Downloads/Yawnly`
- Package root: `com.techmo.yawnly`

## Stack

- **Frontend:** Compose Multiplatform (KMP). Shared code in `composeApp/src/commonMain`.
- **Android entry:** `androidApp` (the Android **application** module). Depends on `composeApp`.
- **iOS entry:** `iosApp` — Xcode host that embeds the Compose framework. **Not a Gradle module.**
- **Backend:** Supabase (`supabase/` — migrations + Edge Functions).
- **Auth + DB:** Supabase. Mobile uses anon key. Service role only inside Edge Functions.
- **DI:** Koin. Modules registered in `KoinInit.android.kt` AND `KoinInit.ios.kt`.
- **Logging:** Napier in shared code. Never `println`.
- **Networking:** Two stacks present.
  - **Supabase SDK** (`io.github.jan-tennert.supabase`) for product APIs. Repositories extend `SupabaseBaseRepository`.
  - **Ktor** + `BaseRepository`/`BaseResponseDto` is legacy skeleton. Do not use for new Yawnly product work.

## Important Feature Packages

- `onboarding`
- `tonightsstory`
- `storygenerator`
- `storyreader`
- `mythology`
- `paywall`
- `profile`
- `sharecard`

## Build Commands

- Shared module compile check: `rtk ./gradlew :composeApp:compileKotlinAndroid`
- Android APK: `rtk ./gradlew :androidApp:assembleDebug`
- iOS: open `iosApp/iosApp.xcodeproj` in Xcode (Gradle runs in the build phase).

First-time setup: copy `appkeys.properties.example` → `appkeys.properties` (gitignored) and fill in Supabase URL, anon key, Google Web Client ID. Signing credentials in `~/.gradle/gradle.properties` or local `credentials.properties`. No CI or linter is configured; compile checks are the verification gate.

## Hard Constraints

- **Supabase is the product backend.** Treat it as the source of truth for product data.
- **No Room for Yawnly product data.** Product data is online-only. Guest data stays in-memory until auth.
- **Ktor/`BaseRepository`/`BaseResponseDto` is legacy skeleton path.** Do not use for new Supabase work.
- **Do not modify** `BaseViewModel`, `BaseRepository`, `BaseResponseDto`, or theme/core infrastructure unless the task explicitly permits it.
- **`core/` must not import `feature/*` types.**
- **Domain models never use `@Serializable`.** Only DTOs do.
- **Layer contract is mandatory:** `ViewModel → UseCase (interface + impl) → Repository (interface) → RepositoryImpl → ApiService`.
- **ApiService is transport-only.** Business decisions belong in use cases.
- **Retry/error strategy** lives at the repository boundary (e.g. `executeSupabaseCall`), not in ApiService.

## What Not To Touch

- Existing `core/` infrastructure: analytics, appEventBus, datastore, db, hapticFeedback, home, ktor, navigation, permissions, systemBarController, userState, webview. Use as-is; extend only when the task says so.
- `BaseRepository` / `BaseResponseDto` — leave intact (Ktor skeleton).
- `BaseViewModel` — extend, don't modify.
- Theme files — add Yawnly colors/fonts alongside, don't delete.

## Project-Specific Deviations from Global Guidelines

- The project uses **two network stacks** (Supabase + Ktor skeleton). The global guideline assumes one. For new Yawnly work, always pick the Supabase stack.
- The project uses SDP/SSP for dimensions (`.sdp`/`.ssp`) and the project tokens `UniversalColor`/`ComposeColor`, `HeightSpacer`/`WidthSpacer`, `BaseActionButton`, `noRippleSafeClick`, `BaseBottomSheet`, `BaseDialog`. Apply the global Compose UI guideline using these specific tokens.

## Backend / Supabase

- Project ID: `ynjiskzaxsqdovmugiwo`
- URL: `https://ynjiskzaxsqdovmugiwo.supabase.co`
- Backend code: `supabase/` (migrations + Edge Functions).
- Keys: `appkeys.properties` (gitignored), accessed via BuildConfig.
- Edge Functions present include OTP login, IAP verification, audio/story generation, payment webhooks. See `repo-map.md` for the current list.

## External TTS / Kokoros

- Kokoros is the self-hosted TTS service for Yawnly. See `projects/kokoros/context.md`.
- Mobile clients must call Supabase Edge Functions, not Kokoros directly.
- Supabase Edge Functions call `https://tts-hetzner.yawnly.org/v1/audio/speech`.
- Required outbound header from Supabase to Kokoros: `X-Yawnly-TTS-Key`.
- Supabase secret name for the raw header value: `HETZNER_TTS_PROXY_KEY`.
- Never store the raw header value in MyWiki, git, app code, or chat.

## Current Release Status

See `current-state.md`.
