# Platform mapping — KMP versus Android-only

> Answers Step 1 of the workflow. Defaults fill gaps only; the project's source sets, modules,
> and pinned artifacts win.

## Contents

1. Detect the target · 2. Where each part lives · 3. API shapes to verify, never assume ·
4. Presentation layout example · 5. Test idiom and per-target reporting

## Detect the target

Read the signals from the settings file, the module build files, and the version catalog — never
from folder names alone. A `commonMain` directory in the tree proves nothing until the
multiplatform plugin is applied and the target is declared.

| Signal | Target |
|---|---|
| `build.gradle*` applies a Kotlin Multiplatform plugin, and a Compose Multiplatform dependency sits in `commonMain` | `kmp shared compose ui` |
| A Kotlin Multiplatform plugin, with an iOS app that renders in SwiftUI or UIKit and consumes a shared framework | `kmp native ios ui` |
| The Android application plugin and Jetpack Compose, with no multiplatform plugin | `android-only` |

Write the matching name on the Screen Profile's `target` line, and the deciding file as its
`evidence`.

One repository can hold both a shared Compose UI and a native iOS UI. That is ambiguity, not a
fourth target: ask which of the two the screen belongs to. Autonomous — record an owner-tagged
open point and follow the majority of the existing screens.

## Where each part lives

Placement is decided by source set and module, not by package name. Each row says which side of
the split a part belongs on; the project's existing split wins over the wording here.

| Concern | KMP with Compose Multiplatform UI | Android-only with Jetpack Compose |
|---|---|---|
| Routes, Screens, UI state, ViewModels | the UI module's `commonMain`, where the APIs support it | the app or feature presentation package or module |
| Domain, use cases, contracts | platform-neutral shared code | the project's existing scheme |
| OS adapters | `androidMain`, `iosMain`, and the other actual targets | the Android platform or integration layer |
| DI | common contracts and factories with platform bindings | manual factories or the existing DI framework |
| Shared reusable UI | the shared UI module | the design-system module |

Domain, use cases, and contracts are designed under `clean-architecture` on both sides.

For a native iOS UI, share the business rules and the contracts, and apply the Route/Screen
pattern to the Compose frontend only. Shared Compose UI is not implied by a shared framework.
Never move Android UI objects into shared code. State and platform integration reach the native
side through the project's own interop design, not through a shape invented here.

## API shapes to verify, never assume

| Need | Android-only usual shape | KMP usual shape | Verify in |
|---|---|---|---|
| ViewModel resolution at the Route | `hiltViewModel()`, or `viewModel()` | the multiplatform ViewModel artifact's `viewModel()` with a factory, or the project's DI | the pinned versions in the version catalog |
| Lifecycle-aware collection | `collectAsStateWithLifecycle()` from the lifecycle-runtime-compose artifact | the multiplatform lifecycle artifact's equivalent, where it is supported | the Lifecycle artifact version |
| Saved state | `SavedStateHandle` | the multiplatform saved-state artifact where it is supported, otherwise a durable store | the saved-state artifact version |

The skill names these as the usual shapes. The project's pinned versions decide whether each
exists; when one does not, use the project's substitute and record it in the Screen Profile.

Record the versions you read on the profile's `pinned artifacts` line and the resolution
mechanism on its `di at the route` line.

## Presentation layout example

```text
features/example/presentation/
  ExampleRoute.kt
  ExampleScreen.kt
  ExampleViewModel.kt
  ExampleUiState.kt
  ExampleIntent.kt
  components/
```

Below the ViewModel — use cases, contracts, data — the layout is `clean-architecture`'s and the
project's; this skill states no home for contracts. Adapt names and source sets to the
repository; never reorganize modules to reproduce this example.

## Test idiom and per-target reporting

Read the project's task list first. Never assert that a task exists.

**Android-only:**

- `./gradlew testDebugUnitTest`, or the project's task — ViewModel intent-to-state tests with
  `kotlinx-coroutines-test`, plus a Flow-testing library when the project already has one.
- Compose UI tests with `createComposeRule()` — the Screen renders from a fixture.
- Instrumented tests for recreation and permission flows, when the screen requests a permission
  or must survive recreation.

**KMP:**

- `./gradlew :shared:allTests`, or the project's shared module and its per-target tasks —
  `jvmTest`, `iosSimulatorArm64Test`, `testDebugUnitTest` — each reported separately.

**Both:**

- A Route integration test for destination scoping and outcome consumption, when the feature
  navigates.
- The project's static and module-boundary checks — lint, detekt, Konsist or ArchUnit where
  present — run through the project's own task.

Report each target's compile, test, and device result on its own line. A green JVM run says
nothing about device behavior or another platform's runtime.
