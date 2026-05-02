---
title: CMP Feature Scaffold
summary: Reusable feature-module scaffold for CMP apps using Clean Architecture, MVVM, Koin, and Compose.
type: guideline
status: canonical
owner: human
created: 2026-05-02
updated: 2026-05-02
last_verified: 2026-05-02
applies_to: [global]
---

# CMP Feature Scaffold

Use this when adding a new feature module to a Compose Multiplatform app.

## Preconditions

Before scaffolding, confirm:
- Feature name and package location.
- Whether the feature needs remote data, local persistence, navigation, analytics, or platform services.
- Which existing components, use cases, repositories, DTOs, route types, and UI tokens can be reused.
- Which platforms are in scope. If one platform is out of scope, record the explicit fallback behavior.

## Standard Folder Shape

```text
feature/<name>/
  data/
    dto/              # transport models, serialization annotations, toDomain()
    network/          # ApiService or transport adapter
    repository/       # RepositoryImpl
    paging/           # only if needed
  domain/
    model/            # plain domain models
    repository/       # repository interfaces
    usecase/          # one interface + impl per business action
  presentation/
    action/           # sealed *Action
    state/            # immutable *State
    sideEffects/      # sealed *SideEffect
    ui/               # *ScreenRoot + private *Screen
    viewmodel/        # BaseViewModel<State, SideEffect>
  di/                 # feature Koin module
  navigation/         # route/nav graph wiring, if navigable
```

Project-specific package names and tokens belong in `projects/<project>/context.md`.

## Layer Responsibilities

- DTOs mirror transport. They may use `@Serializable` and `@SerialName`.
- Domain models are plain Kotlin and never carry serialization annotations.
- Repository interfaces live in domain. Implementations live in data.
- Use cases own business decisions and expose one focused action.
- ViewModels call use cases, reduce state, and emit side effects.
- Composables render state and send actions. They do not call repositories, ApiServices, or business logic directly.

## Registration Checklist

For each new feature:
- Add the feature Koin module and bind interfaces to implementations.
- Register the module in Android and iOS Koin initialization paths required by the project.
- Add navigation route and graph entries if the screen is navigable.
- Add ViewModel tests under the project's primary host-test source set when available.
- Add common tests for pure multiplatform business logic.
- Run the project's compile/test commands from the task packet.

## Platform Code

Use `expect`/`actual` or injected platform services for platform APIs such as file pickers, billing, notifications, media playback, sharing, and native authentication.

Do not leak Android or iOS platform types into `commonMain` domain models. If common code needs platform context, pass a narrow interface or a documented initialization parameter and validate it at the platform boundary.

## Common Mistakes

- Registering a Koin module on Android but not iOS.
- Adding a DTO and using it directly in presentation.
- Calling a repository directly from a ViewModel when the project requires use cases.
- Putting navigation or toast logic in the dumb screen composable.
- Adding raw dimensions, colors, or click handlers instead of project tokens and safe click helpers.
- Letting a platform-specific feature silently default without documenting the out-of-scope platform behavior.
