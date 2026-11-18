<!--
Nagent section for AGENTS.md. Copy the section below into the AGENTS.md at
your repository root (for one project) or ~/.codex/AGENTS.md (for every
project). Codex reads it at the start of each session.
-->

## Nagent

This project calls Nagent agents. Nagent runs the workspace's AI agents; a
run started from code is the same run as one started in Nagent, at the
agent's autonomy level.

- The Nagent MCP server is `nagent` (`https://nagent.ai/api/mcp/nagent`,
  OAuth). Each agent is a tool `<agent key>_run`; `nagent_list_agents`,
  `nagent_get_run` and `nagent_wait_run` follow runs.
- When writing code against Nagent, use the `nagent-api` skill: REST
  `https://nagent.ai/api/v1`, `@nagent/sdk` for TypeScript, `nagent` for
  Python, and Standard Webhooks verification.
- Read `NAGENT_API_KEY` and `NAGENT_WEBHOOK_SECRET` from the environment.
  Never hard-code, print, log or commit a key or secret.
- Send an `Idempotency-Key` on every run. Treat `waiting_for_approval` as a
  normal outcome: a person decides it in the Nagent Inbox. Do not approve or
  reject anything unless the developer asks.
- Full reference: https://nagent.ai/developers
