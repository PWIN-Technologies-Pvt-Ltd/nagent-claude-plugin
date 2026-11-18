# Nagent for OpenAI Codex

Use your Nagent agents from Codex, and let Codex wire the Nagent API, SDKs
and webhooks into your code. Codex connects to the Nagent MCP server
(`https://nagent.ai/api/mcp/nagent`) and signs in with OAuth in your
browser, so there is no key to paste.

You need a Nagent workspace on the **Team** plan or above. Granting access
needs settings permission (an Organisation admin).

## What's inside

`codex/plugins/nagent/` is a plugin in the portable Agent Plugins format,
which Codex and ChatGPT both read:

| Part | What it does |
| --- | --- |
| `mcp.json` | The Nagent MCP server, signed in with OAuth |
| Skill `nagent-setup` | Guided setup: connect, check, then wire the API into your project. Run it with `$nagent-setup` |
| Skill `nagent-run` | Run one agent and show the result: `$nagent-run nora research retail-1.example` |
| Skill `nagent-api` | Loaded when you write code against Nagent: REST, `@nagent/sdk`, `nagent` for Python, idempotency, async runs, errors, webhook verification |

Custom prompts are deprecated in Codex, so the setup and run steps ship as
skills instead of prompt files.

## Install the plugin (Codex CLI, Codex in the ChatGPT desktop app)

1. Add this repository as a plugin marketplace:

   ```bash
   codex plugin marketplace add PWIN-Technologies-Pvt-Ltd/nagent-claude-plugin
   ```

   `codex plugin marketplace list` should show a marketplace named `nagent`.
2. Install the plugin:
   - Codex CLI: enter `/plugins`, pick **Nagent** from the `nagent`
     marketplace and install it. Start a new session afterwards.
   - ChatGPT desktop app: restart it, open **Plugins**, choose the Nagent
     marketplace and install **Nagent**.
3. Sign in when Codex asks (a browser opens on nagent.ai: sign in, pick the
   workspace, allow access). If it does not ask, run `codex mcp login nagent`.
4. Run `$nagent-setup`.

To turn the plugin on for one project only, add this to that project's
`.codex/config.toml`:

```toml
[plugins."nagent@nagent"]
enabled = true
```

## Install by hand (Codex IDE extension, or no plugins)

The Codex IDE extension does not load plugins. Add the MCP server and the
skills yourself; this also works in the CLI.

1. Add the server, either with the CLI:

   ```bash
   codex mcp add nagent --url https://nagent.ai/api/mcp/nagent
   ```

   or by pasting [`config.toml`](config.toml) into `~/.codex/config.toml`.
   That file also raises `tool_timeout_sec` to 300, because an agent run can
   take longer than the 60 second default.
2. Sign in:

   ```bash
   codex mcp login nagent
   ```

   To ask for fewer permissions:
   `codex mcp login nagent --scopes agents:read,agents:run,runs:read`.
   On a machine without a browser, add `--no-browser` and paste the redirect
   URL back into the terminal.
3. Copy the skills into your user skills folder (or `.agents/skills/` in a
   repository):

   ```bash
   git clone --depth 1 https://github.com/PWIN-Technologies-Pvt-Ltd/nagent-claude-plugin nagent-plugin
   mkdir -p ~/.agents/skills
   cp -R nagent-plugin/codex/plugins/nagent/skills/* ~/.agents/skills/
   ```

4. Optional: add the Nagent section from [`AGENTS.md`](AGENTS.md) to your
   repository's `AGENTS.md`, so every Codex session knows the house rules
   (keys from the environment, idempotency keys, approvals left to people).
5. Start a new session and run `$nagent-setup`.

## Using an API key instead of OAuth

For CI or a shared machine, make a key at
[Settings, API & MCP](https://nagent.ai/admin/settings/api), keep it in an
environment variable, and point Codex at that variable:

```toml
[mcp_servers.nagent]
url = "https://nagent.ai/api/mcp/nagent"
bearer_token_env_var = "NAGENT_API_KEY"
```

## A private-label workspace

A workspace on its own domain uses its own host. Use the same steps with
`https://<host>/api/mcp/nagent` as the URL, and a server name such as
`nagent-<name>`.

## Check it works

- `codex mcp list` shows `nagent`.
- In a session, ask "list my Nagent agents". Codex calls
  `nagent_list_agents`.

Runs from Codex are ordinary Nagent runs: the agent works at its autonomy
level, and anything it may not do alone waits for a person in the Nagent
Inbox.

## Where the text comes from

The skill texts are copies. The sources are
`plugins/nagent/skills/nagent-api/` (the Claude plugin's skill) and
`shared/nagent-setup.md` and `shared/nagent-run.md`. Edit those, then run
`python3 scripts/sync-shared.py`.

Docs: [Codex MCP](https://developers.openai.com/codex/mcp),
[plugins](https://developers.openai.com/plugins/build/plugins),
[skills](https://learn.chatgpt.com/docs/build-skills),
[AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md).
