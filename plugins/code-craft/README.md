# Code Craft

[![Latest release](https://img.shields.io/github/v/release/MuhammadBahaa/majarrah-marketplace?display_name=release&label=release&color=brightgreen)](https://github.com/MuhammadBahaa/majarrah-marketplace/releases)
[![License](https://img.shields.io/github/license/MuhammadBahaa/majarrah-marketplace)](https://github.com/MuhammadBahaa/majarrah-marketplace/blob/main/LICENSE)
[![Stars](https://img.shields.io/github/stars/MuhammadBahaa/majarrah-marketplace?style=flat)](https://github.com/MuhammadBahaa/majarrah-marketplace/stargazers)
[![skills.sh installs](https://skills.sh/b/MuhammadBahaa/majarrah-marketplace)](https://skills.sh/MuhammadBahaa/majarrah-marketplace)

Engineering-craft skills for coding agents, by **Majarrah Nexus**.

| Skill | What it does |
|---|---|
| [`clean-architecture`](https://skills.sh/muhammadbahaa/majarrah-marketplace/clean-architecture) | Implements, extends, or refactors a feature with Clean Architecture in whatever stack the project already uses |
| [`compose-screen-architecture`](https://skills.sh/muhammadbahaa/majarrah-marketplace/compose-screen-architecture) | Builds or audits the screen layer of an Android Jetpack Compose or Compose Multiplatform app — Route/Screen split, state ownership, ViewModel lifetimes, outcomes and restoration — and hands everything below the ViewModel to `clean-architecture` |

## What `clean-architecture` does

It enforces one dependency direction —
`frameworks & drivers → adapters → use cases → domain` — and nothing else. It
imposes **boundaries, never tools**: packages, naming, folder shape, state
management, and error idiom are read out of the project, not chosen by the
skill.

The run has four steps:

1. **Detect and profile.** Work out the stack from repo signals, inventory the
   packages already in use by role (DI, HTTP, storage, state, navigation,
   serialization, testing), and study how the project builds features today —
   its own guidance first, then the git history of past features, then the
   code. A role the project has already filled is binding; the skill's defaults
   only apply where the project left one empty.
2. **Propose before code.** Entities, use cases, contracts, data sources,
   mappers, delivery state, wiring, offline policy — plus the layers it plans
   to skip and why. Anything ambiguous stops for your answer.
3. **Build inside-out**, gating each layer before moving outward.
4. **Verify and report honestly**: the project's own tests and
   dependency-direction checks, and every violation found — new or
   pre-existing — reported rather than quietly fixed or hidden.

Bundled stack mappings ship for **Flutter, Android, iOS, React Native, and Node
backends**. Any other stack — web frontend, another backend language, desktop —
is derived live from the project's own conventions against a documented mapping
contract, so there is no per-project setup either way.

Two behaviors are worth knowing before you install it:

- **Existing violations are reported, never copied and never silently
  rewritten.** If the screens around your feature call the API inline, the
  skill will not match that and will not go clean them up.
- **Structural conflicts are your call, not the agent's.** When the project's
  own shape contradicts the layer contract — a repository that is really a use
  case, a missing layer a neighbour absorbed — the skill stops and asks, with
  the cost of each path stated. "Follow the project" is a legitimate answer;
  the point is that it is a decision on the record.

## What `compose-screen-architecture` does

It owns the chain **navigation graph → Route → Screen → components**, with
the ViewModel as the state owner, for Jetpack Compose and Compose
Multiplatform. Three rules hold regardless of project: every piece of state
has one owner and is hoisted no higher than needed; a ViewModel never
retains an Activity, View, launcher, or UI-lifetime adapter, directly or
indirectly; and the project's conventions win over the skill's defaults
while those two rules win over everything.

The run mirrors `clean-architecture`: profile the screen conventions and
pinned artifacts, map ownership before code (state kinds, owners,
lifetimes, destination scope, restoration, outcome consumption, platform
adapters), build inside-out with a gate after each part, then verify and
report **per target** — a green JVM run is never presented as device
proof. It also runs as an audit of an existing screen, returning a findings
table with a proposed fix or an owner-tagged open point per row.

Everything the ViewModel calls — use cases, contracts, data — is
`clean-architecture`'s. XML-to-Compose migration is not this skill.

## Limits

`clean-architecture` is a discipline for writing code, not a verifier of
code you already have: it will not audit a codebase. `compose-screen-architecture`
audits one screen at a time, never a codebase. Both skills' judgments about
your conventions are model judgments. Neither adds packages the project does not
already have. Neither needs an MCP server, network access, or credentials.

## Install

Every route installs the same two skills. Pick the one that matches your agent.

### Codex

```bash
codex plugin marketplace add MuhammadBahaa/majarrah-marketplace
codex plugin marketplace list
codex plugin list
codex plugin add code-craft@majarrah-marketplace
```

`codex plugin list` should now show `code-craft@majarrah-marketplace` as
`installed, enabled`. Start a new Codex task to load both skills.

### Claude Code

```bash
claude plugin marketplace add MuhammadBahaa/majarrah-marketplace
# then, inside Claude Code:
/plugin install code-craft@majarrah-marketplace
```

`/plugin` lists what is installed — `code-craft` should be enabled. Start a new
session to load both skills.

### Cursor, GitHub Copilot, Gemini CLI, and others

```bash
npx skills add MuhammadBahaa/majarrah-marketplace --skill clean-architecture
```

or `--skill compose-screen-architecture`. Drop the `--skill` flag to take
every skill in the marketplace instead. The skills' pages are
[skills.sh/muhammadbahaa/majarrah-marketplace/clean-architecture](https://skills.sh/muhammadbahaa/majarrah-marketplace/clean-architecture)
and
[skills.sh/muhammadbahaa/majarrah-marketplace/compose-screen-architecture](https://skills.sh/muhammadbahaa/majarrah-marketplace/compose-screen-architecture);
each installs into the shared Agent Skills directory these agents read. Gemini CLI
alternative:
`gemini skills install <skill-folder-or-git-url>`. Reload skills or restart the
agent afterwards.

### Manual copy

```bash
git clone https://github.com/MuhammadBahaa/majarrah-marketplace
mkdir -p ~/.agents/skills
cp -r majarrah-marketplace/plugins/code-craft/skills/* ~/.agents/skills/
```

Use `~/.agents/skills/` (user-wide) or `<project>/.agents/skills/` (per
project). Cursor and Copilot also read `~/.claude/skills`, so a Claude Code
install covers them too. Restart the agent afterwards.

## License

[MIT](https://github.com/MuhammadBahaa/majarrah-marketplace/blob/main/LICENSE)
© Majarrah Nexus. No third-party code is vendored into this plugin.
