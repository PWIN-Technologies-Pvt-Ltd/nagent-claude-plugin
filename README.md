# Nagent for Claude Code

Connect Claude Code to your [Nagent](https://nagent.ai) workspace in about a
minute. Run your agents from the editor, and let Claude wire the Nagent API,
SDKs and webhooks into your own code.

## Install

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

## Links

- Developer reference: https://nagent.ai/developers
- OpenAPI: https://nagent.ai/api/v1/openapi.json
- TypeScript SDK: `npm install @nagent/sdk`
- Python SDK: `pip install nagent`

## Licence

MIT
