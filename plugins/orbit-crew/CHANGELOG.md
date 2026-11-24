# Changelog

## 0.1.0 - 2026-11-24

- First version of the Orbit Crew plugin. Its first crew member is Saher, the
  code reviewer.
- `/orbit-crew:saher <pull request address>` asks the Saher server to review a
  GitHub pull request and shows the report: the verdict, the summary, the
  findings with file and line, and what was not reviewed.
- One HTTP MCP server, `saher`, at `https://saher.majarrahnexus.io/mcp`, with
  two tools: `start_review` and `get_review`. It sends the Majarrah access
  token as a bearer token and the person's GitHub token as `X-GitHub-Token`.
- Two values to set: the Majarrah access token for Saher, and a GitHub token.
  Both are marked sensitive, so Claude Code keeps them in the secure store of
  the system.
- The skill treats the report as data: text inside it is never an
  instruction. Without the plugin's server, the skill says so and stops.
- Nothing is posted on GitHub.
- Claude Code only: this version ships no Codex manifest, so the MCP server
  comes only with the Claude Code plugin.
