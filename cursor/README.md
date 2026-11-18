# Nagent for Cursor

Use your Nagent agents from Cursor, and let Cursor's agent wire the Nagent
API, SDKs and webhooks into your code. Cursor connects to the Nagent MCP
server (`https://nagent.ai/api/mcp/nagent`) and signs in with OAuth in your
browser, so there is no key to paste.

You need a Nagent workspace on the **Team** plan or above. Granting access
needs settings permission (an Organisation admin).

## Quick start: add the MCP server

[![Add Nagent MCP server to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/link/mcp/install?name=nagent&config=eyJ1cmwiOiJodHRwczovL25hZ2VudC5haS9hcGkvbWNwL25hZ2VudCJ9)

The button opens Cursor and asks you to install the `nagent` server. The
same link in Cursor's own scheme:

```text
cursor://anysphere.cursor-deeplink/mcp/install?name=nagent&config=eyJ1cmwiOiJodHRwczovL25hZ2VudC5haS9hcGkvbWNwL25hZ2VudCJ9
```

The `config` value is the base64 of `{"url":"https://nagent.ai/api/mcp/nagent"}`.

Then sign in: open **Customize** (or Cursor Settings, MCP), find `nagent`
and choose **Connect**. A browser opens on nagent.ai: sign in, pick the
workspace, allow access. Ask the agent "list my Nagent agents" to check.

The button adds only the MCP server. For the setup and run commands and the
API rule, install the plugin or copy the files as shown below.

## Install the plugin

`cursor/plugins/nagent/` is a Cursor plugin:

| Part | What it does |
| --- | --- |
| `mcp.json` | The Nagent MCP server, signed in with OAuth. It shows as `plugin-nagent-nagent` in approval prompts |
| Command `/nagent-setup` | Guided setup: connect, check, then wire the API into your project. Add `app` or `webhooks` to start at that step |
| Command `/nagent-run` | Run one agent and show the result: `/nagent-run nora research retail-1.example` |
| Rule `nagent-api` | Applied when the agent writes code against Nagent: REST, `@nagent/sdk`, `nagent` for Python, idempotency, async runs, errors, webhook verification |

Until Nagent is listed in the Cursor Marketplace, install it locally:

```bash
git clone --depth 1 https://github.com/PWIN-Technologies-Pvt-Ltd/nagent-claude-plugin nagent-plugin
mkdir -p ~/.cursor/plugins/local
cp -R nagent-plugin/cursor/plugins/nagent ~/.cursor/plugins/local/nagent
```

Copy the folder rather than linking it: Cursor skips a symlink that points
outside `~/.cursor/plugins/local`. Then run **Developer: Reload Window**,
open **Customize** and check that the Nagent rule, commands and MCP server
are there. Connect the server as above, then run `/nagent-setup`.

Teams on Cursor's Teams or Enterprise plan can also import this repository
as a team marketplace (Dashboard, Plugins & MCPs, Add Marketplace, Import
from Repo). The repository carries `.cursor-plugin/marketplace.json` for
that.

## Install by hand

Each part works on its own:

- **MCP server**: put [`mcp.json`](mcp.json) in `.cursor/mcp.json` (this
  project) or `~/.cursor/mcp.json` (every project), or merge its `nagent`
  entry into the file you have.
- **Rule**: copy `plugins/nagent/rules/nagent-api.mdc` into `.cursor/rules/`.
- **Commands**: copy `plugins/nagent/commands/*.md` into `.cursor/commands/`.

## Using an API key instead of OAuth

For a shared machine, make a key at
[Settings, API & MCP](https://nagent.ai/admin/settings/api), keep it in an
environment variable, and reference the variable in `mcp.json`:

```json
{
  "mcpServers": {
    "nagent": {
      "url": "https://nagent.ai/api/mcp/nagent",
      "headers": { "Authorization": "Bearer ${env:NAGENT_API_KEY}" }
    }
  }
}
```

## A private-label workspace

A workspace on its own domain uses its own host. Use the same steps with
`https://<host>/api/mcp/nagent` as the URL and a server name such as
`nagent-<name>`.

Runs from Cursor are ordinary Nagent runs: the agent works at its autonomy
level, and anything it may not do alone waits for a person in the Nagent
Inbox.

## Where the text comes from

The rule and commands are copies. The sources are
`plugins/nagent/skills/nagent-api/` (the Claude plugin's skill) and
`shared/nagent-setup.md` and `shared/nagent-run.md`. Edit those, then run
`python3 scripts/sync-shared.py`.

Docs: [Cursor MCP](https://cursor.com/docs/mcp),
[MCP install links](https://cursor.com/docs/mcp/install-links),
[plugins](https://cursor.com/docs/plugins),
[rules](https://cursor.com/docs/rules).
