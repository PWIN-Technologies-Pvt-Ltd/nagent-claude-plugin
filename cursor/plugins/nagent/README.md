# Nagent plugin for Cursor

Connect Cursor to your [Nagent](https://nagent.ai) workspace. The plugin adds
the Nagent MCP server (`https://nagent.ai/api/mcp/nagent`, signed in with
OAuth in your browser, no key to paste), a `/nagent-setup` command that
connects and then wires the Nagent SDK, an example and a webhook receiver
into your project, a `/nagent-run` command that runs one of your agents, and
a `nagent-api` rule that guides the agent when it writes code against the
Nagent REST API, `@nagent/sdk` or the `nagent` Python package.

After installing, open **Customize**, find the Nagent MCP server and choose
**Connect**. Sign in on nagent.ai, pick the workspace and allow access. Then
run `/nagent-setup`.

You need a Nagent workspace on the **Team** plan or above, and settings
permission (an Organisation admin) to grant access. Runs started from Cursor
are ordinary Nagent runs: anything an agent may not do alone waits for a
person in the Nagent Inbox.

Full setup guide:
https://github.com/PWIN-Technologies-Pvt-Ltd/nagent-claude-plugin/tree/main/cursor

Privacy policy: https://nagent.ai/privacy-policy

Licence: MIT
