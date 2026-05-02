---
title: Compose Multiplatform Architecture
summary: Reusable CMP rules — module roles, layer flow, DTO/domain boundaries, Koin, core-to-feature direction.
type: guideline
status: canonical
owner: human
created: 2026-05-02
updated: 2026-05-02
last_verified: 2026-05-02
applies_to: [global]
---

# Compose Multiplatform Architecture

Reusable rules for any CMP project. Project-specific exceptions stay in `projects/<project>/context.md`.

## Mental Model

CMP project layout:
- `composeApp/` — Kotlin Multiplatform **library** (not the Android application). Almost all shared code lives here.
- `commonMain` — shared Kotlin, Compose, ViewModels, use cases, repositories. Default for new code.
- `androidMain` / `iosMain` — `actual` implementations and platform services (TTS, DataStore init, billing, share sheets).
- `androidApp/` — Android **application** module: `Application`, `MainActivity`, manifest. Depends on `composeApp`.
- `iosApp/` — iOS Xcode host that embeds the Compose framework. Typically NOT a Gradle module.

## Layer Direction

```
Presentation → Domain ← Data
```

- Domain never imports `data` or `presentation`.
- `core/` must not import `feature/*` types. If `core` needs a feature capability, define an interface in `core` and implement it in the feature module.

## Required Layer Flow

```
ViewModel
  → UseCase (interface + Impl)
    → Repository (interface)
      → RepositoryImpl
        → ApiService
```

- **ApiService is transport-only.** No business decisions, no entitlement resolution, no retry/backoff policies, no cross-table orchestration.
- **UseCase owns business rules.** Pure domain logic.
- **RepositoryImpl owns data composition and mapping.** Map DTOs to domain. Coordinate sources. Keep transport details out of use cases.
- **Retry / error strategy lives at the repository boundary** (e.g. an `executeSupabaseCall`-style helper), not in ApiService.

Any violation above is a blocking architecture finding (P1) in code review.

## DTO ↔ Domain Separation

- DTOs live under `feature/<name>/data/dto/`. They use `@Serializable` and `@SerialName(...)` and provide a `toDomain()` mapper.
- Domain models live under `feature/<name>/domain/model/`. They are plain data classes with no serialization annotations.
- Never serialize a domain model. Never expose a DTO to presentation.

## Koin Dependency Injection

- Each feature owns `feature/<name>/di/<Feature>Module.kt`.
- Bind interfaces to implementations. Use `viewModel { ... }` for ViewModels.
- **Register every Koin module in both `KoinInit.android.kt` and `KoinInit.ios.kt`.** Missing one platform causes runtime DI failures.
- The shared client singleton (e.g. Supabase) lives in a shared module (e.g. `sharedModule`).

## Feature Folder Structure

```
feature/<name>/
  data/
    dto/           # @Serializable + @SerialName + toDomain()
    network/       # ApiService (Supabase or Ktor)
    repository/    # *RepositoryImpl
  domain/
    model/         # plain data classes, no @Serializable
    repository/    # repository interfaces
    usecase/       # interface + *Impl per action
  presentation/
    action/        # sealed *ScreenAction
    state/         # *ScreenState data class
    sideEffects/   # sealed *ScreenSideEffect
    ui/            # *ScreenRoot + private *Screen
    viewmodel/     # extends BaseViewModel<State, SideEffect>
  di/              # <name>Module.kt
  navigation/      # NavGraphBuilder.<Name>Navigation
```

Build order: domain → data → presentation → di → navigation.

## Core-to-Feature Rule

`core/` provides infrastructure (DataStore, navigation primitives, base classes, theme tokens). It must remain feature-agnostic. If a core component needs a capability that a feature owns:
1. Declare an interface in `core/`.
2. Implement it in the feature module.
3. Bind the implementation in the feature's Koin module.

## Logging

Use a multiplatform logger (e.g. `Napier`) in shared code. Never `println` in `commonMain`.

## Build Verification

A CMP project should at minimum compile shared code and assemble the Android app.

Examples:

```shell
rtk ./gradlew :composeApp:compileKotlinAndroid
rtk ./gradlew :androidApp:assembleDebug
```

iOS is verified by opening the Xcode project; Gradle runs in the Xcode build phase.

## Do/Don't

- **Do** keep `composeApp` as a KMP library and `androidApp` as the application entry point.
- **Do** put new code in `commonMain` unless platform `expect/actual` is genuinely required.
- **Do** map DTO → domain at the repository boundary.
- **Don't** import `data` or `presentation` types from `domain`.
- **Don't** put business rules in ApiService or in the UI layer.
- **Don't** modify shared base infrastructure (`BaseViewModel`, base repositories, theme/core) inside a feature task — extend instead.
