---
name: compose-screen-architecture
description: "Use when a Compose screen — Android Jetpack Compose or Compose Multiplatform (KMP) — is created or wired into navigation, or when an existing Compose screen must be reviewed or refactored for state ownership, lifecycle safety, or navigation effects. Symptoms count too — a ViewModel holding an Activity, Context, launcher, or navigation controller; the same value editable in both Screen and ViewModel; navigation that repeats after rotation or process death; a Screen resolving its own ViewModel or DI; loading, error, and saving as loose Booleans; permission or picker flows tangled with business state. Owns the navigation graph → Route → Screen chain and the ViewModel's state contract. Below the ViewModel (use cases, contracts, data) is clean-architecture; XML-to-Compose migration is not this skill. Project conventions are detected first and always win."
---

# Compose screen architecture (Android and KMP, adapts to the project)

Build and audit the screen layer — navigation graph → Route → Screen → UI components, with
the ViewModel as the state owner — on Android Jetpack Compose and on Compose Multiplatform.
The skill imposes ownership and lifetime **boundaries**, never tools: the DI mechanism, the
navigation library, the intent style, and the scaffold and preview rules are read from the
project, not chosen here.

This file is a **router**. You **must** open and follow the reference each step links —
never work from memory of it.

**Not this skill:** everything below the ViewModel — use cases, repository and service
contracts, data implementations, error translation, offline policy, composition-root wiring
of non-UI dependencies — is `clean-architecture`; this skill restates only the one
`clean-architecture` rule the screen layer enforces at its own gates (the ViewModel calls use
cases, never a repository or data source) and adds nothing below the ViewModel. Also outside
it: XML-to-Compose migration; design-system or component-library work; a visual tweak that
changes no state, lifetime, or navigation behavior; finding a defect's root cause. If
`clean-architecture` is not installed, say so in the Step 4 report and design nothing below the
ViewModel.

## The three iron rules

1. **One owner per state.** Every piece of state has one authoritative owner, named before
   code, and is hoisted no higher than its readers, writers, and business logic require. No
   separately editable copies of the same value live in the Screen and the ViewModel;
   derived values are computed from the authoritative input, never stored beside it.
2. **Lifetimes point one way.** A dependency is valid for the full lifetime of its consumer.
   A ViewModel never retains an Activity, View, permission launcher, navigation controller,
   or UI-lifetime adapter — not directly, and not through a gateway, closure, state holder,
   or factory. Creating the adapter in the Route does not fix a ViewModel that then holds it.
3. **Convention precedence, contract floor.** The project's convention wins over this
   skill's defaults — sealed intent versus focused callbacks, DI mechanism, scaffold and
   preview rules, file naming. Rules 1 and 2 and `clean-architecture`'s dependency direction
   win over everything, including existing code that violates them: violations are
   **reported, never copied, never silently rewritten**. When the project's own screen shape
   contradicts a rule and the new screen must sit on it, **stop and ask the operator**
   (Step 2, *Structural conflicts*).

## Workflow

### Step 1 — Detect the platform and profile the screen conventions

Read, in this order: the project's own guidance (project skills, agent instructions,
architecture docs) → one existing feature screen end to end (Route, Screen, ViewModel,
UiState, graph entry) → the pinned Compose, Lifecycle, ViewModel, and navigation artifact
versions → the DI mechanism → the intent style. Decide whether the target is KMP (shared
Compose UI, or native iOS UI with shared business code) or Android-only. Fill the **Screen
Profile** block. An ambiguous target or two conflicting screen conventions → ask; autonomous
→ record an open point and proceed with the dominant convention.
→ [`references/platform-mapping.md`](references/platform-mapping.md).

```
## Screen Profile
target: <kmp shared compose ui | kmp native ios ui | android-only>   evidence: <file>
example screen studied: <Route/Screen/ViewModel paths>
pinned artifacts: compose <v> · lifecycle <v> · viewmodel <v> · navigation <v>   (from <file>)
di at the route: <hilt | koin | manual factories | other>   evidence: <manifest line>
intent style: <sealed intent | focused callbacks>   example: <path>
scaffold / preview rules: <what the project does>   example: <path>
divergences observed (not copied, not rewritten): <list or none>
structural conflicts raised → operator's decision: <conflict → "project style" | "iron rule" | "unresolved", or none>
```

### Step 2 — Map ownership before code

Present, scaled to the screen:

- for every piece of state: its **kind** (persisted domain data / editable draft / derived
  display value / local visual state), **owner**, **lifetime**, **destination scope**, and
  **restoration requirement**
- for every outcome (a completed save, an authentication result): how it is modeled in
  state, who consumes it, and the consumption policy that keeps navigation idempotent
- for every platform interaction (permission, settings prompt, biometrics, picker): which
  adapter, bound where, and how denial, cancellation, and recreation are handled
- the **use cases the ViewModel will call** — named here, designed under `clean-architecture`
- the parts skipped and why: a static screen takes navigation callbacks and has no
  ViewModel; a plain state-holder class for UI coordination is fine

If anything is ambiguous, or the operator asked to see the plan first, stop for their answer;
autonomous → record an owner-tagged open point and proceed with the dominant convention.
→ [`references/state-ownership.md`](references/state-ownership.md) and
[`references/route-screen-contract.md`](references/route-screen-contract.md).

#### Structural conflicts — ask, don't decide alone

Convention precedence settles idiom, not ownership and lifetime. You are in a structural
conflict when:

- the project's ViewModels take a `Context`, an `Activity`, or a launcher by convention
- Screens resolve their own ViewModel (`viewModel()` / `hiltViewModel()` inside the rendering
  composable) throughout
- navigation is driven from inside ViewModels through a held controller
- a graph-shared ViewModel is the default for every destination
- ViewModels call repositories directly by convention

Raise it as one question, before writing code:

> `<part>` here does `<what it actually does>`, which breaks `<the rule>`.
> Follow the project's style — consistent with its neighbours, no churn, the divergence stays?
> Or give this screen `<the correct shape>` — matches the rule, but this corner stops looking
> like the rest of the codebase?

Both answers are legitimate; record the answer in the Screen Profile. With no operator, or no
answer, build to the iron rules and record the conflict as unresolved. Ask only when the
screen being built has to sit on the conflict; a divergence elsewhere is a line in the report.

### Step 3 — Build inside-out, gating each part

UiState and Intent → ViewModel → Screen with a deterministic preview fixture → Route → graph
entry. Gate each part before moving to the next:

- **after UiState and Intent:** every state the screen can actually be in is modeled, and none
  it cannot be in — loading / content / empty / failure / saving / completion as states, never
  as unrelated Booleans; the state type is immutable; outcomes that must survive process death are reconstructible from
  durable data or saved state, never from in-memory state alone
- **after the ViewModel:** it exposes an immutable state stream with private mutation; it
  imports no Activity, View, Context, launcher, navigation controller, Compose runtime, or
  data implementation; it calls **use cases only**; duplicate submissions are guarded; the
  draft is kept on a recoverable failure
- **after the Screen:** it takes state and callbacks and resolves no ViewModel and no DI; it
  renders from a supplied fixture in a preview or test; local visual state (scroll, focus,
  animation) stays in the components that own it
- **after the Route:** the ViewModel is resolved with the intended destination owner; state
  is collected with the destination's lifecycle, never an accidental app-global owner;
  platform adapters are created and bound to the current UI lifetime and their results
  forwarded as intents; each navigation outcome is consumed exactly once
- **after the graph entry:** it passes destination arguments and navigation callbacks only,
  and back or direct UI navigation reaches the callback without the ViewModel

→ Shapes: [`references/route-screen-contract.md`](references/route-screen-contract.md).

### Step 4 — Verify and report per target

Run the project's unit tests, its Compose UI tests where they exist, its static checks, and
its module-boundary checks; the commands come from the platform mapping. Report each target's
compile, test, and device result **separately** — passing JVM tests are never presented as
proof of device behavior or of another platform's runtime. Report every ownership or lifetime
violation found, new or pre-existing, and every structural conflict raised together with the
operator's decision, using this shape:

```markdown
### Verification: <screen>

| Target | Check | Result |
|---|---|---|
| <android \| jvm \| ios simulator \| device> | <unit \| ui \| static \| module-boundary \| instrumented> | passed / failed — <detail> / not run — <reason> |
```

One row per target and check; violations and structural conflicts follow the table, one line each.
→ Commands: [`references/platform-mapping.md`](references/platform-mapping.md).

## Audit entry

A request to review, audit, or refactor an existing screen enters here. Run Step 1, walk the
Step 3 gates against the existing code, then return the findings table:

```markdown
### Screen audit: <screen>

| # | Finding | Rule | Where | Action |
|---|---|---|---|---|
| 1 | <what is wrong> | <iron rule or gate> | <file:line> | fix applied / proposed fix / open point — <owner> |
```

A review returns proposed fixes; fixes are applied only when the request asks for a refactor.
An empty audit reads `none`, never an empty table.

## Red flags — stop, you are about to break a boundary

- A `Context`, `Activity`, launcher, or `NavController` parameter or field on a ViewModel.
- `viewModel()` / `hiltViewModel()` / a DI lookup inside a rendering composable.
- A Boolean "navigated" flag as the only guard against repeated navigation.
- A `Channel` or shared flow carrying an outcome that must survive process death.
- The same field editable in both a `remember` block and the UiState.
- A ViewModel calling a repository or a data source.
- Collecting state with the Activity's lifecycle from inside a nested destination.
- Adding a navigation, DI, or lifecycle library the project does not already have.

| Rationalization | Reality |
|---|---|
| "The launcher is only used once, the ViewModel can hold it" | A UI-lifetime object outlives nothing; after recreation the ViewModel holds a dead launcher. Bind it in the Route, forward the result as an intent. |
| "A `navigated` Boolean is enough" | It is not exactly-once: it resets on process death and races with restoration. Define consumption or acknowledgement and make the navigation idempotent. |
| "It's a simple read, the ViewModel can call the repository" | `clean-architecture`'s rule: a thin read gets one slim use case. The ViewModel never names a contract. |
| "Every screen here resolves its own ViewModel — I'm matching the project" | Convention precedence covers idiom, not lifetime and ownership violations. Raise it as a structural conflict. |
| "One graph-shared ViewModel keeps things simple" | It also keeps every destination's state alive together. A shared scope is a deliberate decision recorded in the ownership map, not a default. |
| "The unit tests pass, so it works on device" | Report per target. JVM tests prove state transitions, not lifecycle, recreation, or the other platform's runtime. |
| "Restoration can wait until we see a bug" | Restoration is decided at design time: which values are saved state, which are durable, which are recomputed. Retrofitting it is the bug. |

## Quick reference

**You must use this table as the gate check** at every gate in Step 3, in the audit, and in
the Step 4 confirmation.

| Part | Owns or does | Must keep out |
|---|---|---|
| Composition root | creates dependencies and factories, binds contracts, chooses scopes and lifetimes | rendering and business decisions |
| Navigation graph | destinations, arguments, back-stack behavior, navigation callbacks | screen business logic, persistence |
| `FeatureRoute` | ViewModel resolution, lifecycle-aware state collection, callback wiring, platform UI coordination, outcome consumption | rendering detail, business rules, storage calls |
| `FeatureViewModel` | screen state, intent handling, UI-state production, operation outcomes; calls use cases | Activity / View / Context / launcher references, navigation controllers, repositories, data sources |
| `FeatureScreen` | renders supplied state and emits callbacks | DI lookup, ViewModel access, use cases, persistence |
| UI components | render values and callbacks; keep local visual state when nothing else on the screen needs to control it | screen-level business dependencies |
| Platform UI adapter | UI-lifetime permissions, settings prompts, biometrics, document pickers | retention by a longer-lived ViewModel |
| Below the ViewModel | use cases, contracts, data, error translation — see `clean-architecture` | — |

## References

**You must use this table:** open the listed file when its step runs — never substitute
memory for it.

| File | Use it for |
|---|---|
| [`references/platform-mapping.md`](references/platform-mapping.md) | Step 1 · KMP versus Android-only placement, source sets, ViewModel and Lifecycle API shapes, DI at the Route, test idiom and per-target reporting |
| [`references/state-ownership.md`](references/state-ownership.md) | Step 2 · the four kinds of state, hoisting, UiState modeling, outcomes and consumption, effects, restoration, saved state versus durable storage |
| [`references/route-screen-contract.md`](references/route-screen-contract.md) | Steps 2–3 · Route duties, the Screen API shape, ViewModel resolution and destination scope, lifetime rules, platform UI adapters |

## Modes

Interactive by default: ask the one question that unblocks a real convention conflict.
Autonomous when the caller announces it (a driving agent or unattended run): never ask;
record each gap or conflict as an owner-tagged open point in the returned result and proceed with the detected convention.

For a structural conflict with no answer, "proceed" means build to the iron rules and record the conflict as unresolved in the Screen Profile.

## Tested

2026-09-08: cold trigger test, eight prompts judged from the description alone, 8/8 as expected;
three pressure runs on Sonnet 5 — a launcher offered to the ViewModel, a Boolean navigation flag
under process death, a direct repository read — all GREEN by inspection of the produced code.
