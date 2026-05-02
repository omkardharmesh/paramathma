---
title: Compose UI Conventions
summary: Root/Content split, state/action/effect pattern, token-first dimensions and colors, tap handlers.
type: guideline
status: canonical
owner: human
created: 2026-05-02
updated: 2026-05-02
last_verified: 2026-05-02
applies_to: [global]
---

# Compose UI Conventions

Reusable Compose UI rules. Project brand details (color palette, typography choices, animation flavor) live in `projects/<project>/context.md` or `decisions.md`, not here.

## Root / Content Split

Each screen splits into two composables:

- `*ScreenRoot` — **smart**. Holds the ViewModel, collects state, runs side effects, and dispatches navigation.
- `*Screen` — **dumb**. Receives `state` and `onAction` only. No ViewModel reference. No navigation.

```kotlin
@Composable
fun FooScreenRoot(navController: NavController) {
    val viewModel: FooViewModel = koinViewModel()
    val state by viewModel.state.collectAsStateWithLifecycle()

    LaunchedEffect(key1 = viewModel) {
        viewModel.sideEffects.collect { effect -> /* handle */ }
    }

    FooScreen(state = state, onAction = viewModel::onAction)
}

@Composable
private fun FooScreen(state: FooState, onAction: (FooAction) -> Unit) { /* ... */ }
```

## State / Action / Side Effect

- `*ScreenState` — immutable data class. The single source of truth that the dumb composable renders.
- `*ScreenAction` — sealed interface for every user intent.
- `*ScreenSideEffect` — sealed interface for one-shot effects (navigation, snackbar, dialog trigger).

Side effects are collected inside `LaunchedEffect(key1 = viewModel)` in the Root composable. Never collect them inside content composables.

## ViewModel Conventions

- Extend the project's `BaseViewModel<State, SideEffect>`.
- Inject via `koinViewModel()` in the Root composable.
- Expose state through a state holder (e.g. `StateFlow`) and consume with `collectAsStateWithLifecycle()`.
- The ViewModel calls use cases. It does not call repositories or ApiServices directly.

## Tokens, Not Raw Numbers

Use design tokens for every visual decision. Avoid raw `.dp`, `.sp`, raw color literals, or raw `Spacer()`.

- Spacing — use the project's spacer/dimension tokens (e.g. `.sdp`, `.ssp`, named spacers).
- Colors — use the project's color tokens (e.g. `UniversalColor`, `ComposeColor`), never raw hex in screen code.
- Typography — use the project's `MaterialTheme.typography` mapping.
- Spacing components — use the project's spacing primitives (e.g. `HeightSpacer`, `WidthSpacer`), not bare `Spacer(Modifier.height(8.dp))`.

If a screen needs a value that no token provides, the right move is to add a token, not to inline a literal.

## Tap Handlers

- Use the project's debounced/safe click helper for every tap (e.g. `noRippleSafeClick`, `safeClickable`). Do not call raw `clickable { ... }` on primary CTAs.
- Use the project's reusable button component (e.g. `BaseActionButton`) for primary CTAs. Reusable bottom sheets and dialogs (`BaseBottomSheet`, `BaseDialog`) for modals.

## Top-Level Screen Composition Checklist

For every new screen:
- [ ] `*ScreenRoot` exists and owns `koinViewModel()`, state collection, and side effect collection.
- [ ] `*Screen` exists, is `private`, takes `state` + `onAction` only.
- [ ] `*ScreenState` is a single immutable data class.
- [ ] `*ScreenAction` is a sealed type covering every user intent.
- [ ] `*ScreenSideEffect` is sealed.
- [ ] All dimensions use tokens.
- [ ] All colors use tokens.
- [ ] All taps go through the safe click helper.

## Animation Caution

- Animation must be purposeful (acknowledge an action, smooth a navigation transition, communicate state change).
- Avoid broad automatic motion (looping global ambient animations, bouncing decorations) unless a project decision approves it for that specific screen.
- Prefer the project's existing motion tokens and easing curves over inventing new ones per screen.
