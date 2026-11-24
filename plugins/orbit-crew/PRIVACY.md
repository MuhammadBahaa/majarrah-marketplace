# Orbit Crew Privacy

The plugin itself has no code that runs on your machine. It holds settings, one MCP server entry, and one skill. It sends no telemetry. Claude Code saves your two tokens in the secure store of your system.

Today the crew has one member, Saher, the code reviewer. When you ask Saher for a review, Claude Code calls the Saher server at `https://saher.majarrahnexus.io`. The server is run by Majarrah Nexus.

## What the server receives

| Value | Why |
| --- | --- |
| The pull request address | To know which pull request to review |
| Your Majarrah access token | To check that you may use Saher, and to count your limits |
| Your GitHub token | To read the pull request from GitHub |

## What the server does with it

| Point | Detail |
| --- | --- |
| Reading the code | The server reads the pull request from GitHub with your GitHub token. It sees only what your token can see. |
| Writing the review | The server gives the pull request to an AI model, Claude, with the Anthropic API key of Majarrah Nexus. The model writes the report. |
| Your GitHub token | Kept in memory for the review only. Not written to a file, and removed from every log line. |
| The report | Kept in the server's memory for one hour after the review ends, so the same commit gives the same report. Then it is deleted. |
| Money count | The server saves how much each access token spends in a day, by the token's id, to apply the day limit. |
| Logs | One line for each review: the review id, the pull request (`owner/repo#number`), the token's id, the result, and the cost. No token is in it. |
| GitHub | Nothing is posted on GitHub. |

## Claude Code

Claude Code shows the report in your session. That part runs under the terms and privacy policy of Claude Code itself.

## Questions

Ask Majarrah Nexus, who gave you your access token.
