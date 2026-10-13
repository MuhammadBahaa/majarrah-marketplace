---
name: clean-architecture
description: Use when a feature in any codebase — mobile app (Flutter, Android, iOS, React Native, KMP), web frontend, backend service, desktop app — must be implemented, extended, or refactored with Clean Architecture (layers, use cases, repositories, gateways, Uncle Bob structure), or when the work spans delivery (screens or endpoints), business rules, and data (API, database, cache) together. Its symptoms count as the same need — business logic piling up in widgets, screens, components, view models, controllers, or route handlers; UI or endpoints calling HTTP clients, databases, or storage directly; API, ORM, or database models leaking across layers; a feature needing offline support or sync. The project's stack, packages, and conventions are detected first and always win over this skill's defaults.
---

# Clean Architecture (any stack, adapts to the project)

Implement features with Clean Architecture — dependency direction
`frameworks & drivers → adapters → use cases → domain` — in whatever stack and idiom the
project already uses: mobile app, web frontend, backend service, or desktop. The skill
imposes **boundaries**, never tools: packages, naming, and state management are detected
from the project, not chosen by the skill.

This file is a **router**. You **must** open and follow the reference each step links —
never work from memory of it.

**Not this skill:** pure visual tweaks with no data or business change; finding the root
cause of a defect, or deciding where an already-understood fix belongs; documentation-only
work; comparing an implementation against a design file.

## The two iron rules

1. **Dependency direction.** Source dependencies point inward only. The delivery layer — a
   screen, widget, state holder, controller, or route handler — never reaches past use cases
   into databases, storage, HTTP clients, or platform APIs; a use case never touches delivery,
   navigation or routing, or vendor types; the domain imports nothing from outside itself.
2. **Convention precedence.** The project's existing convention beats this skill's defaults —
   naming, state management, DI, folders, error idiom. The boundaries in rule 1 beat
   everything, including existing code that violates them: violations are **reported, never
   copied, never silently rewritten**. When the convention *is* the violation — the project's
   own shape for a layer contradicts rule 1 — neither answer is yours to pick alone:
   **stop and ask the operator** (Step 2, *Structural conflicts*).

## Workflow

### Step 1 — Detect the stack and profile the project

Detect the stack from repo signals, then — before any code — inventory the packages by role
(DI, HTTP, storage, state management, navigation, serialization, testing) and **study how the
project already builds features**, in order: the project's own guidance (project skills, agent
instructions, architecture docs) → the git history of past features → your own search of the
code. Fill the **Project Profile** block with what the study found and where it came from.
**A filled role is binding:** if the project already has an injection framework, an HTTP
client, or a storage package, the feature relies on it — the mapping's defaults exist only
for roles the project leaves empty.
Ambiguous stack or two conflicting conventions → ask the operator, never guess.
→ Follow [`references/stack-detection.md`](references/stack-detection.md); load the bundled
mapping from [`references/stack-mappings/`](references/stack-mappings/) (flutter · android ·
ios · react-native) or derive one live per its contract.

### Step 2 — Propose before code

Present, scaled to the feature's size: the Project Profile, then the planned design —

- entities and value objects; use cases (one per user operation)
- repository and gateway contracts; data sources (remote/local), models, mappers,
  repository implementations
- delivery: UI state (explicit loading / content / empty / error), state holder, navigation
  outcomes — or, on services: endpoints/handlers, request/response DTOs, error-to-status mapping
- DI registrations at the composition root
- offline / lifecycle / permission concerns — for offline: source of truth, cache lifetime,
  read/write policy, optimistic updates, retry, conflict resolution, sync visibility
- **layers skipped and why** (skip test: no meaningful business boundary — a recorded
  decision, never a silent shortcut)
- divergences observed in existing code (reported, not copied, not rewritten)

State the dependency direction across these parts. If anything is ambiguous, or the operator
asked to see the plan first, stop for their answer; otherwise implement exactly what was
proposed.

#### Structural conflicts — ask, don't decide alone

Convention precedence exists for cosmetic differences: an `*Interactor` suffix, a `ui/` folder
instead of `presentation/`. It was never meant to settle cases where the project's own shape
contradicts the layer contract. There, "follow the project" and "follow the contract" produce
visibly different code, and the operator is the one who lives with the result — so the choice
is theirs, not yours.

You are in one of these cases when:

- a layer's role is played by a type named for another role — a repository named `*UseCase`,
  a "use case" that is really a data source
- a layer is absent and a neighbour absorbed its work — no repository at all; state holders or
  screens calling clients directly
- a layer's code sits in a package named for a different layer — HTTP calls and model
  conversion under `domain/`
- an inner contract is typed in outer types — DTOs, ORM entities, or framework types in domain
  signatures
- the error idiom carries vendor exceptions across a boundary that is supposed to translate them

Raise it as one question, before writing code, with the cost of each path stated plainly:

> `<layer>` here is `<what it actually is>`, which breaks `<the rule>`.
> Follow the project's style — consistent with its neighbours, no churn, the divergence stays?
> Or introduce `<the correct shape>` for this feature — matches the contract, but this corner
> stops looking like the rest of the codebase?

Both answers are legitimate, and **"follow the project" is a perfectly good decision** — the
point is that the operator makes it knowingly instead of the skill quietly assuming it. Record
the answer in the proposal so the next person can see it was a choice rather than an oversight.
If the operator declines to decide — or no operator can answer in this run — build this
feature to the layer contract and record the conflict as an unresolved divergence.

Keep the bar meaningful: ask when the feature you are building has to sit on top of the
conflict. A divergence elsewhere in the codebase that this feature never touches is a line in
the report, not a question.

### Step 3 — Build inside-out

Domain → use cases → data (models, sources, mappers, implementations, error translation) →
delivery (state holders, UI state, and screens — or controllers, DTOs, and responses) →
composition-root wiring. Each layer's test first, then the layer, in
the project's own test idiom (testing matrix in the layer contract). The test's first
failure must come from the behavior it checks, never from code that does not exist yet:
after the test, give the layer its bare shape — types and signatures, no behavior — run the
test and see it fail on its own check, then write the behavior. The bare shape is not the
layer, so the test still comes first. Gate each layer before moving out:

- after domain + use cases: banned-import check is clean; business rules sit in domain
  objects, use cases only orchestrate
- after data: no API/DB model type escapes the layer; technical errors are translated to
  application errors at this boundary
- after delivery: the state holder or controller calls **use cases only**; UI state is
  immutable with explicit loading / content / empty / error, and service responses map
  application errors to transport codes
- after wiring: only the composition root knows concrete types

→ Rules: [`references/layer-contract.md`](references/layer-contract.md) · shapes: the stack
mapping from Step 1.

### Step 4 — Verify and report honestly

- Run the project's unit tests (commands in the stack mapping).
- Run the mapping's dependency-direction checks (or the project's own arch-test tooling).
- Confirm: presentation calls only use cases; domain and use cases have no mobile-framework,
  database, or network imports.
- Report: every use case, contract, adapter, mapper, and registration created; test results
  as they actually are; **every dependency violation found — new or pre-existing — reported,
  never hidden or bypassed**; and every structural conflict raised, with the decision the
  operator made and which shape the code now follows.

## Red flags — stop, you are about to violate a boundary

- Typing an HTTP-client, database, ORM, or storage import inside a screen, widget, state
  holder, controller, or route handler.
- A state holder, UI state, or API response typed with raw JSON maps / ORM rows instead of
  entities and mapped models.
- Catching a vendor exception (`DioException`, `IOException`, `URLError`, axios/ORM errors)
  in delivery code.
- Writing cache/offline logic inside a state holder or screen.
- Registering nothing in the composition root because "the state holder can just build it".
- Adding a state-management, DI, HTTP, or storage package the project didn't already have.

| Rationalization | Reality |
|---|---|
| "The existing screens call the API inline — I'm matching the project" | Convention precedence covers idiom, not violations. Divergences are reported, never copied. |
| "There's no repository here, so my use case can just call the big client interface" | A missing or misnamed layer is a structural conflict, not a style choice. Put it to the operator before you build on it. |
| "Renaming their god-interface is out of scope, so I'll say nothing and move on" | Scope is a fine reason not to *rewrite* it; it is not a reason not to *ask*. Raise it, let them choose, record the answer. |
| "Small feature — layers are over-engineering" | Maybe — run the skip test and record the skips in the proposal. Boundaries (direction, model conversion, error translation) hold at any size. |
| "I'll mock the HTTP client to test the state holder" | Needing an HTTP mock in a presentation test *is* the violation signal: the state holder should take use cases. |
| "The JSON map already has exactly the fields the UI needs" | Until the first backend rename, which then breaks screens directly. Convert at the data edge. |
| "Caching in the state holder is just a few lines" | Those lines are your offline policy — hidden where nobody can find or test it. It belongs in the repository, stated in the proposal. |
| "I'll wire DI later, calling the singleton is faster now" | Later never comes; inner layers now know concretes. Wiring is part of the feature, at the composition root. |

## Quick reference

**You must use this table as the gate check:** at every layer gate in Step 3, and again in
the Step 4 confirmation, check the finished layer against its row — what it holds, what it
may know, what it must never know. The full rules live in
[`references/layer-contract.md`](references/layer-contract.md); this table is their fast form.

| Layer | Holds | May know | Never knows |
|---|---|---|---|
| domain | entities, value objects, invariants, domain errors, domain services | itself | UI, state mgmt, HTTP, storage, platform, ORM/DTO types |
| use cases | one operation each, constructor-injected | domain + repository/gateway contracts | UI, navigation, HTTP, DB, repository impls, data sources, models, vendor exceptions |
| data | repository & gateway contracts + their impls, sources, models, mappers, error translation | domain | presentation |
| delivery (presentation) | screens, state holders, immutable UI state, navigation — or controllers, handlers, response mapping | use cases + domain types | repository impls, data sources, models, network/storage/ORM |
| composition root | all wiring | everything | — |

## References

**You must use this table:** it maps each workflow step to the file to open at that step.
Open the listed file when its step runs — never substitute memory for it.

| File | Use it for |
|---|---|
| [`references/stack-detection.md`](references/stack-detection.md) | stack signals, package/convention inventory, precedence, Project Profile, mapping contract |
| [`references/layer-contract.md`](references/layer-contract.md) | the full per-layer rules, offline policy points, error translation, testing matrix, skip test |
| [`references/stack-mappings/flutter.md`](references/stack-mappings/flutter.md) · [`android.md`](references/stack-mappings/android.md) · [`ios.md`](references/stack-mappings/ios.md) · [`react-native.md`](references/stack-mappings/react-native.md) · [`node-backend.md`](references/stack-mappings/node-backend.md) | per-stack shapes, role hints, banned-import checks, composition root, test commands, greenfield fallbacks — other stacks derive live |
