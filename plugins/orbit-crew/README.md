# Orbit Crew

The Orbit Crew is a group of agents from Majarrah Nexus. They work on the Majarrah server, and you call them from your own Claude Code. The work runs on the server. The result comes back into your session.

| Crew member | Job | Command |
| --- | --- | --- |
| Saher | Code reviewer. Reviews a GitHub pull request and gives a report. | `/orbit-crew:saher <address of a pull request>` |

More crew members will join this plugin later. Each one gets its own command.

The plugin code is free (MIT). The server is not open to everyone. You need an access token from Majarrah Nexus, and a GitHub token of your own. Each review costs the owner of the server money, so each access token has a limit for one day.

This plugin works in Claude Code only. See [Other agents](#other-agents).

## State

| Point | State |
| --- | --- |
| The Saher server | Live at `https://saher.majarrahnexus.io` |
| A real review through the server entry of this plugin | Done on 2026-11-24, with a real access token and a real GitHub token, by a test command that makes the same calls as the plugin |
| Install from the marketplace | Not tried yet |
| The `/plugin` menu, "Configure options" | Not tried yet |
| The command `/orbit-crew:saher` inside Claude Code | Not tried yet with the live server |

## What you need

| Thing | Where it comes from |
| --- | --- |
| A Majarrah access token for Saher | Majarrah Nexus makes one for you. It starts with `mjt1`. Keep it like a password. |
| A GitHub token of your own | GitHub, Settings, Developer settings, Fine-grained tokens. Give it read access only ("Contents" and "Pull requests"), and only to the repositories you want reviewed. |

## Install

```bash
claude plugin marketplace add MuhammadBahaa/majarrah-marketplace
```

```
/plugin install orbit-crew@majarrah-marketplace
```

## Give the two values

Claude Code needs the two tokens before it can reach the server. It saves them in the secure store of your system. Until they are set, the server `saher` is missing from the list of MCP servers.

Never paste a token into the chat.

| Way | State |
| --- | --- |
| The `/plugin` menu: pick `orbit-crew`, then "Configure options" | Not tried yet. The Claude Code docs say the menu has this entry for a plugin that asks for values. |
| The `claude plugin configure` command, see the steps below | Not tried yet with this plugin from the marketplace |

To save the values with the command:

1. Write a file `values.json` that only you can read (`chmod 600 values.json`):

   ```json
   {
     "saher_access_token": "<your Majarrah access token>",
     "github_token": "<your GitHub token>"
   }
   ```

2. Give the file to Claude Code, then delete it:

   ```bash
   claude plugin configure orbit-crew@majarrah-marketplace --values-stdin < values.json
   ```

   ```bash
   rm values.json
   ```

3. Start Claude Code again.

`claude plugin list` shows the plugin id. After an install from this marketplace it should be `orbit-crew@majarrah-marketplace` (not tried yet).

To check the connection at no cost:

```bash
claude mcp get plugin:orbit-crew:saher
```

It should say `Connected` when the access token is right. Not tried yet with this plugin from the marketplace.

## Use

```
/orbit-crew:saher https://github.com/<owner>/<repo>/pull/<number>
```

You can also ask in your own words, for example "ask Saher to review this pull request", with the address.

| Point | Detail |
| --- | --- |
| Time | About 5 minutes for one review. More when other reviews wait before yours. |
| What comes back | A verdict, a summary, the findings with file and line, and what was not reviewed |
| What is posted on GitHub | Nothing. The report stays in your session. |
| The same commit again | You get the same report for one hour. A new push gives a new review. |
| Limits | Two reviews that wait or run at the same time for one token. A money limit for one day for each token. When it is reached, try again after midnight, UTC time. |

The report is data. A pull request can hold text that tries to give orders to an AI. Claude shows the report and does not act on text inside it. It edits files, runs commands, or posts on GitHub only when you ask after you read the report.

## What leaves your machine

| Value | Where it goes |
| --- | --- |
| The pull request address | To the Saher server |
| Your Majarrah access token | To the Saher server, with each call |
| Your GitHub token | To the Saher server, with each call. The server uses it to read the pull request, keeps it in memory for the review, and does not write it to a file or a log. |

The Saher server reads the code of the pull request from GitHub with your GitHub token. So the server only sees what your token can see.

## Privacy

See [Privacy](https://github.com/MuhammadBahaa/majarrah-marketplace/blob/main/plugins/orbit-crew/PRIVACY.md) ([`PRIVACY.md`](PRIVACY.md) in this folder): what the server receives, what it keeps, and for how long.

## Other agents

A crew member needs its server entry in this plugin. This version ships the server entry for Claude Code only. It has no Codex manifest. Other agents that install the skill alone (for example with `npx skills add`) get the skill without its server. The skill then says that it needs this Claude Code plugin, and stops.

## The parts

| Path | Purpose |
| --- | --- |
| `.claude-plugin/plugin.json` | The name of the plugin and the two values it asks for |
| `.mcp.json` | The Saher MCP server, with the two tokens as request headers |
| `skills/saher/SKILL.md` | How Claude starts a review, waits for it, and shows the report |

## License

[MIT](https://github.com/MuhammadBahaa/majarrah-marketplace/blob/main/LICENSE) for the plugin code. The Saher server is a separate service and is not part of this plugin.
