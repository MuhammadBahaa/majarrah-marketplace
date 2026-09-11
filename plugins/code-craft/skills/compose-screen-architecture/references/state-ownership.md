# State ownership — who owns what, and what survives what

> Stack-neutral for Compose on Android and KMP. The project's convention chooses intent style
> and names; these rules say what must be true regardless.

## Contents

1. The four kinds of state · 2. Hoisting · 3. Modeling the UiState · 4. Intents and callbacks ·
5. Outcomes · 6. Effects and transient actions · 7. Restoration ·
8. Multi-step saves at the screen

## The four kinds of state

| Kind | Examples | Usual owner | Survives |
|---|---|---|---|
| Persisted domain data | a saved profile, an order | domain and data, reached through use cases | process death |
| Editable draft | the form fields being typed | the ViewModel's UiState; a small draft also in saved state | configuration change; process death only if saved |
| Derived display value | a formatted total, a validation message | computed from the authoritative input, never stored beside it | recomputed |
| Local visual state | scroll position, focus, animation, an expanded card | the component, via `remember` / `rememberSaveable` | per the project's saving rules |

One authoritative owner per piece of state: a value with two writers is a defect to fix in the
ownership map, not in code.

## Hoisting

Hoist each value to the lowest owner that satisfies its readers, its writers, and the business
logic that acts on it. Hoisting does not mean every value belongs in a ViewModel or a Route.
Scroll position, focus, animation, and component interaction state stay near their UI when
nothing else on the screen needs to control them.

App-level navigation state and complex UI coordination may live in a plain state-holder class
instead. A ViewModel must not retain such a holder if the holder contains shorter-lived UI
objects — retaining it makes the ViewModel hold objects that are already gone.

## Modeling the UiState

The screen state type is immutable and exposed as a state stream; the mutable side and every
transition stay private to the ViewModel. Model loading, empty and content, failure, saving, and
completion only where each is meaningful — never as unrelated Booleans, which admit combinations
the screen can never be in and leave every reader guessing which flag wins.

Avoid — unrelated Booleans:

```kotlin
data class State(
    val isLoading: Boolean,
    val isError: Boolean,
    val isSaved: Boolean,
    val draft: Draft,
)
```

Prefer — one sealed state where every case is reachable:

```kotlin
sealed interface State {
    object Loading : State
    data class Content(val draft: Draft, val saveStatus: SaveStatus) : State
    data class Failed(val error: ScreenError, val draft: Draft) : State
}
```

`Failed` carries the draft on purpose: a recoverable failure must never lose what the user
typed, so the draft rides through the failure state and back into `Content` on retry.

## Intents and callbacks

State goes down, events come up. Express the events as a sealed intent type or as focused
callbacks — whichever the project already uses. Neither style requires an MVI framework, and
neither is introduced here to displace the other.

A business intent may be wired straight to a ViewModel method reference; a lambda that only
forwards adds nothing. Back and other direct UI navigation invoke the navigation callback
without passing through the ViewModel.

## Outcomes

An **outcome** is a result of the screen's work that something outside the screen must act on: a
completed save, an authentication result, a deletion that ends the destination. Model it in
observable state. The Route observes it and invokes the navigation callback.

Every outcome needs a **consumption policy** — who acknowledges it, when, and what proves it was
consumed — defined together with navigation that is idempotent, so a lifecycle restart or a
restoration never navigates twice. A Boolean flag alone is not an exactly-once guarantee.

One workable shape:

1. The outcome carries an identity, or the state holds a single `pendingNavigation` value.
2. The Route navigates, then calls `onNavigated(id)`.
3. The ViewModel clears that outcome and ignores an acknowledgement it has already cleared.

An outcome that must survive process death is reconstructed from durable domain data or from the
minimum persisted marker. In-memory state alone is insufficient.

## Effects and transient actions

Lifecycle-aware collection does not by itself make a navigation or platform effect
lifecycle-safe: the state arrives safely and the effect still runs against a destination that is
no longer current. Gate those actions to the active destination state.

A `Channel` or a shared flow is acceptable for a transient action only after three questions are
answered explicitly — is loss or replay acceptable, who consumes the action, and when is it
cancelled. Never use one for a durable outcome: a value that must survive process death cannot
live in a stream that dies with the process.

## Restoration

| Event | What survives by default | What you must do |
|---|---|---|
| Recomposition | everything — the ViewModel, `remember`, `rememberSaveable` | nothing |
| Configuration change (Android) | the ViewModel and `rememberSaveable`; plain `remember` is lost | put whatever a rotation must keep in the ViewModel or in saved state |
| Back-stack removal | nothing screen-scoped — the ViewModel and its saved state are cleared; durable storage remains | persist whatever the user should find again on re-entry |
| Process death | only saved state and durable storage | save the small values; reconstruct the rest from durable data |

Use saved state for small restorable values — route arguments, an unfinished draft's fields. Use
durable storage for data that must outlive the screen or the process. Recompute derived values
on restore instead of serializing an object graph to carry them across.

## Multi-step saves at the screen

The screen owns what a user can see and do while a save runs:

- **Duplicate submission is guarded.** A `Saving` status disables the trigger, and repeat
  intents that arrive anyway are ignored, not queued.
- **The draft survives a partial failure.** The failure state keeps the draft and names an error
  the screen can render usefully.
- **Retry and cancellation are intents**, modeled like any other event rather than hidden inside
  the save call.

Atomicity, transactions, and recovery policy belong to the data layer — see `clean-architecture`
and its offline and data rules. A new use case alone does not make two writes atomic.
