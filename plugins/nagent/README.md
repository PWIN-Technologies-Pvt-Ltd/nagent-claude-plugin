# Nagent plugin for Claude

Connect Claude to your [Nagent](https://nagent.ai) workspace. Run your
Nagent agents from Claude, follow their runs, and let Claude wire the Nagent
API, SDKs and webhooks into your own code.

## What's inside

| Part | What it does |
| --- | --- |
| MCP connector `nagent` | `https://nagent.ai/api/mcp/nagent`, signed in with OAuth in your browser, no key to paste. Each agent you can reach becomes a tool `<agent>_run`, plus `nagent_list_agents`, `nagent_get_run`, `nagent_wait_run` and, where your access allows, knowledge, leads, triggers and approvals tools |
| `/nagent:setup` | Guided setup: connect, check, then wire the API into your project |
| `/nagent:run <agent> <ask>` | Run one agent and show the result |
| Skill `nagent-api` | Loaded when you write code against Nagent: REST, `@nagent/sdk`, `nagent` for Python, idempotency, async runs, errors, webhook verification |

## Install in Claude Code

```text
/plugin marketplace add PWIN-Technologies-Pvt-Ltd/nagent-claude-plugin
/plugin install nagent@nagent
/nagent:setup
```

Setup signs you in (OAuth in your browser), lists the agents you can reach
and, if you want, sets up the SDK, an example and a webhook receiver in the
current project.

## Requirements

A Nagent workspace on the **Team** plan or above. Granting access needs
settings permission (an Organisation admin).

Runs from Claude are ordinary Nagent runs: the agent works at its autonomy
level, and anything it may not do alone waits for a person in the Nagent
Inbox.

## Privacy

Nagent's privacy policy: https://nagent.ai/privacy-policy. The plugin itself
stores nothing; the connector sends your requests to your Nagent workspace
under the access you grant at sign-in, and you can revoke that access in
Nagent at any time.

## Links

- Developer reference: https://nagent.ai/developers
- Setup for other apps (Codex, Cursor, ChatGPT, Claude chat):
  https://github.com/PWIN-Technologies-Pvt-Ltd/nagent-claude-plugin

## Licence

MIT
