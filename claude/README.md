# Nagent in Claude (claude.ai and the Claude desktop app)

Run your Nagent agents from a Claude conversation, follow their runs, and
see what waits for a person's approval. Claude connects to the Nagent MCP
server (`https://nagent.ai/api/mcp/nagent`) as a custom connector and signs
in with OAuth, so there is no key to paste.

You need a Nagent workspace on the **Team** plan or above. Granting access
needs settings permission (an Organisation admin).

For Claude Code, use the plugin instead: see the
[main README](../README.md) or [`plugins/nagent`](../plugins/nagent).

## Add Nagent as a custom connector

Connectors you add on claude.ai also appear in the Claude desktop app and
the mobile apps, because they belong to your Claude account. The desktop
app does not read remote servers from `claude_desktop_config.json`; add
them here.

### Free, Pro, Max or Ultra plan

1. Open [Customize, Connectors](https://claude.ai/customize/connectors) (in
   the desktop app: **Customize**, **Connectors**).
2. Select **Add**, then **Add custom connector**.
3. Name: `Nagent`. Server URL: `https://nagent.ai/api/mcp/nagent`. Click
   **Continue**.
4. Authentication: keep the options Claude marks **Detected** (sign in with
   OAuth). For **OAuth client**, choose **Register automatically**. Nagent
   does not yet support Claude's published identity (Client ID Metadata
   Documents), so that option fails today. Leave the client ID, secret and
   request headers empty.
5. Click **Add**, then **Connect**. A browser window opens on nagent.ai:
   sign in, pick the workspace and allow access.

On the Free plan you can add one custom connector.

### Team or Enterprise plan

An Owner adds the connector once for the organisation, then each member
connects with their own Nagent account.

1. Owner: open
   [Organization settings, Connectors](https://claude.ai/admin-settings/connectors),
   select **Add**, then **Custom**, and choose **Web** if asked.
2. Enter the name and URL as above, keep the **Detected** authentication
   settings, choose **Register automatically**, and click **Add**.
3. Members: open **Customize**, **Connectors**, find Nagent (labelled
   **Custom**) and click **Connect**.

## Use it

In a conversation, select **+**, then **Connectors**, and make sure Nagent
is on. Then ask, for example:

- "List my Nagent agents and their autonomy levels."
- "Ask nora to research retail-1.example and draft a first email."
- "What is run `<run id>` waiting on?"

You can set each Nagent tool to always allow, ask, or block on the
connector's page under **Tool permissions**.

Runs from Claude are ordinary Nagent runs: the agent works at its autonomy
level, and anything it may not do alone waits for a person in the Nagent
Inbox.

## Known limits

- claude.ai and the desktop app accept tool results up to about 150,000
  characters and wait up to 240 seconds per tool call. A very large
  workspace's agent list can exceed that today; ask for a specific agent by
  key when you know it. Long agent runs come back as `running`; ask Claude
  to check the run again.
- Desktop extensions (`.mcpb`) are for servers that run on your own
  computer. Nagent is a hosted server, so use the custom connector.

## A private-label workspace

A workspace on its own domain uses its own host. Use the same steps with
`https://<host>/api/mcp/nagent` as the server URL.

## When Nagent is in the directory

Once Nagent is listed in the Claude Connectors Directory, connect it from
the directory under **Customize**, **Connectors** instead of adding it by
URL. The
Nagent plugin in this repository (connector, skills and commands) can also
be listed in the Claude plugin directory for claude.ai and Cowork.
[`SUBMISSION.md`](SUBMISSION.md) tracks what both listings need.

Docs: [Add a connector that isn't in the directory](https://claude.com/docs/connectors/custom/add-unlisted),
[Authentication for connectors](https://claude.com/docs/connectors/building/authentication).
