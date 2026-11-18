# Nagent for Claude Code and other AI apps

Connect your AI app to your [Nagent](https://nagent.ai) workspace in about a
minute. Run your agents from the editor or a chat, and let the assistant
wire the Nagent API, SDKs and webhooks into your own code.

Every app connects to the same Nagent MCP server,
`https://nagent.ai/api/mcp/nagent`, and signs in with OAuth in your browser,
so there is no key to paste.

## Use Nagent in

| App | What you get | Setup |
| --- | --- | --- |
| Claude Code | Plugin: MCP server, `/nagent:setup`, `/nagent:run`, `nagent-api` skill | [Below](#install-in-claude-code) |
| Claude chat (claude.ai and the desktop app) | Custom connector | [claude/](claude/README.md) |
| OpenAI Codex (CLI, IDE extension, ChatGPT desktop app) | Plugin: MCP server, `nagent-setup`, `nagent-run` and `nagent-api` skills | [codex/](codex/README.md) |
| Cursor | "Add to Cursor" link, plugin with `/nagent-setup`, `/nagent-run` and a `nagent-api` rule | [cursor/](cursor/README.md) |
| ChatGPT | Custom MCP server now; directory listing to follow | [chatgpt/](chatgpt/README.md) |

Directory submissions: [ChatGPT and Codex](chatgpt/SUBMISSION.md),
[Claude](claude/SUBMISSION.md).

## Install in Claude Code

```text
/plugin marketplace add PWIN-Technologies-Pvt-Ltd/nagent-claude-plugin
/plugin install nagent@nagent
```

Then:

```text
/nagent:setup
```

It walks you through signing in (OAuth in your browser, no key to paste),
lists the agents you can reach, and, if you want, sets up the SDK, an example
and a webhook receiver in the current project.

You need a Nagent workspace on the **Team** plan or above. Granting access
needs settings permission (an Organisation admin).

## What's inside

| Part | What it does |
| --- | --- |
| MCP server `nagent` | `https://nagent.ai/api/mcp/nagent`, signed in with OAuth. Each agent you can reach becomes a tool `<agent>_run`, plus `nagent_list_agents`, `nagent_get_run`, `nagent_wait_run` and, where your access allows, knowledge, leads, triggers and approvals tools |
| `/nagent:setup` | Guided setup: connect, check, then wire the API into your project |
| `/nagent:run <agent> <ask>` | Run one agent and show the result |
| Skill `nagent-api` | Loaded when you write code against Nagent: REST, `@nagent/sdk`, `nagent` for Python, idempotency, async runs, errors, webhook verification |

Runs from Claude Code are ordinary Nagent runs: the agent works at its
autonomy level, and anything it may not do alone waits for a person in the
Nagent Inbox.

## Using an API key instead of OAuth

For CI or a shared machine, make a key at
[Settings, API & MCP](https://nagent.ai/admin/settings/api) and add the
server by hand:

```bash
claude mcp add --transport http nagent https://nagent.ai/api/mcp/nagent \
  --header "Authorization: Bearer $NAGENT_API_KEY"
```

## One source of truth

The `nagent-api` skill lives in `plugins/nagent/skills/nagent-api/`. The
setup and run steps for Codex and Cursor live in `shared/`. The Codex
skills, the Cursor rule and commands, and the icon and licence copies are
generated from those by `python3 scripts/sync-shared.py`; run it with
`--check` to confirm nothing is out of date. The Claude Code commands in
`plugins/nagent/commands/` are the Claude-specific version of the same
steps: change both when the steps change.

## Links

- Developer reference: https://nagent.ai/developers
- OpenAPI: https://nagent.ai/api/v1/openapi.json
- TypeScript SDK: `npm install @nagent/sdk`
- Python SDK: `pip install nagent`

## Licence

MIT
