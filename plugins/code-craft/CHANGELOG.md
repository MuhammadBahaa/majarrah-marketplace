# Changelog

Public release history for Code Craft. Versions before 0.13.0 were private and
are not listed here; the version number is shared with the private plugin, so
the first public entry starts at 0.13.0 rather than 1.0.0.

## 0.15.0 - 2026-09-08
- New skill, `compose-screen-architecture`: the screen layer for Android
  Jetpack Compose and Compose Multiplatform. It owns the navigation graph →
  Route → Screen → components chain and the ViewModel as state owner: the
  Route/Screen split, one owner per piece of state, ViewModel scope and
  lifetimes (never an Activity, launcher, or UI adapter inside a ViewModel),
  outcomes that survive process death, exactly-once navigation, restoration,
  platform UI adapters, and KMP versus Android-only placement.
- Everything below the ViewModel — use cases, contracts, data — hands off to
  `clean-architecture`; the two skills never contradict each other.
- Same discipline as `clean-architecture`: the project's conventions are
  detected first and always win; violations are reported, never copied;
  where the project's own screen shape contradicts a rule, the skill stops
  and asks rather than deciding alone.
- Also runs as an audit of an existing Compose screen, returning a findings
  table.
- `clean-architecture` is unchanged except for one pointer in its Android
  mapping.

## 0.13.0 - 2026-08-10
- First public release. Ships one skill, `clean-architecture`: Clean
  Architecture feature implementation for any stack — mobile (Flutter, Android,
  iOS, React Native), web frontend, backend service, or desktop.
- Stack, packages, and conventions are detected from the project and always win
  over the skill's defaults, so there is no per-project setup. Bundled mappings
  ship for Flutter, Android, iOS, React Native, and Node backends; every other
  stack is derived live against a documented mapping contract.
- Four-step workflow: detect and profile the project, propose the design before
  writing code, build inside-out with a gate at each layer, then verify against
  the project's own tests and dependency-direction checks.
- Published under MIT, the same terms as the rest of this marketplace.
- Installs on Codex (`codex plugin add code-craft@majarrah-marketplace`),
  Claude Code, and any agent reading the Agent Skills directory via
  `npx skills add` or a manual copy.
- Pre-existing dependency violations are reported, never copied and never
  silently rewritten. Where the project's own shape contradicts the layer
  contract, the skill stops and asks rather than deciding alone.
