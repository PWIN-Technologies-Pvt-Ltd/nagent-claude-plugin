# Nagent in ChatGPT

Run your Nagent agents from a ChatGPT conversation, follow their runs, and
see what waits for a person's approval. ChatGPT connects to the Nagent MCP
server (`https://nagent.ai/api/mcp/nagent`) and signs in with OAuth, so
there is no key to paste.

You need a Nagent workspace on the **Team** plan or above. Granting access
needs settings permission (an Organisation admin).

## Connect Nagent as a custom MCP server

Nagent is not in the ChatGPT plugin directory yet, so add it by URL. This
works in ChatGPT on the web; your account or workspace settings decide
whether custom MCP servers are allowed. In a workspace, ask an admin if the
option is missing.

1. Go to [ChatGPT Plugins](https://chatgpt.com/plugins).
2. Select the plus button, then **Add custom MCP server**.
3. Name: `Nagent`. Description (optional): `Run my Nagent agents and follow
   their runs.`
4. Under **Connection**, enter the server URL:
   `https://nagent.ai/api/mcp/nagent`
5. Authentication: **OAuth**. Leave the client ID and secret empty, so
   ChatGPT registers itself with Nagent automatically.
6. Review the risk warning, select **I understand and want to continue**,
   then **Create as a plugin**.
7. Install the new Nagent plugin from your personal plugins. When ChatGPT
   asks you to connect, a browser window opens on nagent.ai: sign in, pick
   the workspace and allow access.

## Use it

Start a new conversation, type `@` and pick **Nagent**. For example:

- "List my Nagent agents and their autonomy levels."
- "Ask nora to research retail-1.example and draft a first email."
- "What is run `<run id>` waiting on?"

ChatGPT asks you to confirm each tool call that is not marked read-only.
You can remember your choice for the rest of a conversation.

Runs from ChatGPT are ordinary Nagent runs: the agent works at its autonomy
level, and anything it may not do alone waits for a person in the Nagent
Inbox. ChatGPT does not decide approvals unless you ask it to and your
access allows it.

## Codex in the ChatGPT desktop app

To use Nagent while coding, install the Nagent plugin for Codex instead. It
adds the setup, run and API skills as well as the MCP server. See
[`../codex/README.md`](../codex/README.md).

## A private-label workspace

A workspace on its own domain uses its own host. Use the same steps with
`https://<host>/api/mcp/nagent` as the server URL.

## Known limits

- A very large workspace returns a long agent list. Ask for a specific
  agent by key when you know it.
- Agent runs that take several minutes come back as `running`. Ask ChatGPT
  to check the run again a little later.

## When Nagent is in the directory

Once Nagent is published in the plugin directory shared by ChatGPT and
Codex, search for **Nagent** under [Plugins](https://chatgpt.com/plugins),
install it and connect. The directory listing replaces the custom server.
[`SUBMISSION.md`](SUBMISSION.md) tracks what that listing needs.

Docs: [Add custom MCP server](https://developers.openai.com/api/docs/guides/custom-mcp-server),
[Connect and test your plugin](https://developers.openai.com/plugins/deploy/connect-chatgpt).
