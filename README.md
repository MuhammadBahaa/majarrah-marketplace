# Majarrah Nexus — Marketplace

<img width="1280" height="640" alt="majarrah-marketplace-social-preview" src="https://github.com/user-attachments/assets/ef5e2d8f-4c39-410b-a85e-1647dd658b41" />

[![License](https://img.shields.io/github/license/MuhammadBahaa/majarrah-marketplace)](LICENSE)
[![Stars](https://img.shields.io/github/stars/MuhammadBahaa/majarrah-marketplace?style=flat)](https://github.com/MuhammadBahaa/majarrah-marketplace/stargazers)
[![skills.sh installs](https://skills.sh/b/MuhammadBahaa/majarrah-marketplace)](https://skills.sh/MuhammadBahaa/majarrah-marketplace)

Free, open AI-agent skills from **Majarrah Nexus**. Works with Codex, Claude
Code, and every agent that supports the Agent Skills open standard — including
Cursor, GitHub Copilot, and Gemini CLI.

<!--
  Generated from the private Majarrah Cubiq monorepo
  (distribution/majarrah-marketplace/README.md). Edits made here are
  overwritten on the next release -- change the source instead.
  This notice is an HTML comment so it does not render on the public page.
-->

## Plugins

| Plugin | What it does |
|--------|--------------|
| [`skill-craft`](plugins/skill-craft) | Technical review, guided walkthroughs, and a human-attention approval gate for agent skills, slash commands, and plugins. |
| [`code-craft`](plugins/code-craft) | Clean Architecture feature implementation for any stack, and the Compose screen layer for Android and KMP. |

Every skill also has its own page on skills.sh:
[`skill-craft-review`](https://skills.sh/muhammadbahaa/majarrah-marketplace/skill-craft-review) ·
[`skill-walkthrough`](https://skills.sh/muhammadbahaa/majarrah-marketplace/skill-walkthrough) ·
[`skill-approval-gate`](https://skills.sh/muhammadbahaa/majarrah-marketplace/skill-approval-gate) ·
[`clean-architecture`](https://skills.sh/muhammadbahaa/majarrah-marketplace/clean-architecture) ·
[`compose-screen-architecture`](https://skills.sh/muhammadbahaa/majarrah-marketplace/compose-screen-architecture)

## Install

Every route installs the same skills. Pick the one that matches your agent.

### ChatGPT

[Install Skill Craft from the ChatGPT Plugins Directory](https://chatgpt.com/plugins/plugins_6a8c5b59d1d88191b45fc86f7cfb92e0),
then select **Install plugin**.

### Codex

```bash
codex plugin marketplace add MuhammadBahaa/majarrah-marketplace
codex plugin marketplace list
codex plugin list
codex plugin add skill-craft@majarrah-marketplace
codex plugin add code-craft@majarrah-marketplace
```

`codex plugin list` should now show `skill-craft@majarrah-marketplace` and
`code-craft@majarrah-marketplace` as `installed, enabled`. Start a
new Codex task to load the skills.

### Claude Code

```bash
claude plugin marketplace add MuhammadBahaa/majarrah-marketplace
# then, inside Claude Code:
/plugin install skill-craft@majarrah-marketplace
/plugin install code-craft@majarrah-marketplace
```

`/plugin` lists what is installed — `skill-craft` and `code-craft` should both
be enabled. Start a new session to load the skills.

### Cursor, GitHub Copilot, Gemini CLI, and others

```bash
npx skills add MuhammadBahaa/majarrah-marketplace
```

[skills.sh](https://skills.sh/MuhammadBahaa/majarrah-marketplace) installs into
the shared Agent Skills directory these agents read. Gemini CLI alternative:
`gemini skills install <skill-folder-or-git-url>`. Reload skills or restart the
agent afterwards.

## License

[MIT](LICENSE) © Majarrah Nexus
