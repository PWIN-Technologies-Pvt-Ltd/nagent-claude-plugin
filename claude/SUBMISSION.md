# Anthropic directory: submission packet for Nagent

Anthropic's directory takes two kinds of submission, and Nagent should make
both, paired with each other:

1. **MCP connector**: the hosted server `https://nagent.ai/api/mcp/nagent`.
   Listed in the Connectors Directory for claude.ai, the desktop and mobile
   apps, Claude Code and Cowork.
2. **Plugin**: `plugins/nagent` in this repository (connector, the
   `nagent-api` skill, and the setup and run commands). In claude.ai chat a
   command loads as a skill.

Items marked **OWNER** need a decision or material from the Nagent owner.
Items marked **SERVER** need a change in the nagent-website repository.
Everything else is ready.

Sources: [Submit a connector](https://claude.com/docs/connectors/building/submission),
[Connector pre-submission checklist](https://claude.com/docs/connectors/building/review-criteria),
[Authentication for connectors](https://claude.com/docs/connectors/building/authentication),
[Build an MCP server for Claude](https://claude.com/docs/connectors/building/index),
[Plugin pre-submission checklist](https://claude.com/docs/plugins/pre-submission-checklist),
[Publish to the directory](https://claude.com/docs/directory/publish).

## 1. Who submits

| Item | Status |
| --- | --- |
| A paid Claude plan with access to the developer portal at https://claude.ai/directory/manage | **OWNER** |
| Agreement to the [Software Directory Terms](https://support.claude.com/en/articles/13145338-anthropic-software-directory-terms) and [Policy](https://support.claude.com/en/articles/13145358-anthropic-software-directory-policy) | **OWNER** |

## 2. Connector requirements

| Requirement | Status |
| --- | --- |
| Remote server over HTTPS | Ready |
| OAuth 2.0 for an authenticated service | Ready: DCR, PKCE `S256`, refresh tokens rotate, `invalid_grant` on a spent refresh token |
| `401` with `WWW-Authenticate: Bearer resource_metadata=...`; `resource` equals the server URL; one authorization server listed | Ready (checked 18 Nov 2026) |
| Redirect `https://claude.ai/api/mcp/auth_callback` for claude.ai, desktop, mobile and Cowork; any-port loopback (`localhost` and `127.0.0.1`) for Claude Code | Ready: Nagent accepts any `https` redirect and loopback on any port |
| Every tool has a `title` and the applicable `readOnlyHint` or `destructiveHint` | **SERVER**: no tool has annotations today, and only `<agent>_run` has a title. See the table below |
| Tool names 64 characters or fewer | **SERVER** check: `<agent key>_run` must stay within 64 characters for long agent keys |
| Tool results within about 150,000 characters (claude.ai and desktop) and calls within 240 seconds | **SERVER**: `nagent_list_agents` returns about 153,000 characters for a big workspace, which fails on claude.ai. Long agent runs should return `running` well before 240 seconds |
| Least-privilege consent | **SERVER** (recommended): Claude requests every scope in `scopes_supported` (17) unless the `401` challenge carries a `scope` parameter. Add `scope="agents:read agents:run runs:read"` |
| Client registration that scales | **SERVER** or **OWNER**: DCR makes a new client on every fresh connection. Anthropic recommends Client ID Metadata Documents (advertise `client_id_metadata_document_supported: true` with `none` auth) or an Anthropic-held client: a confidential client created in Nagent with redirect `https://claude.ai/api/mcp/auth_callback`, whose ID and secret the owner enters in the portal |
| Clear setup and usage documentation | Ready: [`claude/README.md`](README.md) and https://nagent.ai/developers. **OWNER**: publish a public help page for the Claude connector by the publish date |
| Privacy policy URL | https://nagent.ai/privacy-policy. **OWNER** confirm it covers data collection, use and storage, third-party sharing, retention and contact |

Proposed annotations (**SERVER** to add; **OWNER** to confirm the marked
ones). Claude runs read-only tools without a prompt and always prompts for
destructive ones.

| Tool | Title | readOnlyHint | destructiveHint |
| --- | --- | --- | --- |
| `<agent key>_run` | Run `<agent name>` | false | true (**OWNER**: high-autonomy agents can send or publish without a person) |
| `nagent_list_agents` | List agents | true | false |
| `nagent_get_run` | Get a run | true | false |
| `nagent_wait_run` | Wait for a run | true | false |
| `nagent_read_smriti` | Read agent memory | true | false |
| `nagent_list_proposals` | List SEO and AEO proposals | true | false |
| `nagent_decide_approval` | Decide an approval | false | true |
| `nagent_search_knowledge` | Search knowledge | true (**OWNER**: semantic mode spends credits) | false |
| `nagent_read_knowledge` | Read a knowledge document | true | false |
| `nagent_add_knowledge` | Add a knowledge document | false | false |
| `nagent_list_leads` | List leads | true | false |
| `nagent_get_lead` | Get a lead | true | false |
| `nagent_create_lead` | Create a lead | false | false |
| `nagent_update_lead` | Update a lead | false | true |
| `nagent_list_triggers` | List triggers | true | false |
| `nagent_fire_trigger` | Fire a trigger | false | true |
| `nagent_subscribe_event` | Create an event trigger | false | false |

Policy points for **OWNER** before submitting:

- Anthropic does not accept connectors that generate images, video or audio
  through AI models, or that move money. Nagent agents that make images or
  voice, or finance agents in a workspace, run through `<agent>_run`.
  Decide how to describe this in **Use cases** and **Data handling**, and
  whether the listing should exclude those agents.
- Tool descriptions must not tell Claude how to behave. The server's
  descriptions read as plain descriptions today; keep it that way.

## 3. Portal steps for the connector

| Step | What to enter | Status |
| --- | --- | --- |
| 1. Connection | `https://nagent.ai/api/mcp/nagent`, one URL for everyone. Private-label hosts need **Users connect to different URLs**, which can publish only after Verified review; leave them out of the first listing | Ready |
| 2. Tools | Synced from the server; fix flagged titles and annotations first | **SERVER** |
| 3. Listing | Name: Nagent. One-liner (up to 200 characters): "Run your Nagent agents, follow their runs and approvals, and work with your workspace's knowledge and leads." Description (up to 2,000): see below. Categories (1 to 5): **OWNER**. Documentation URL: https://nagent.ai/developers. Privacy policy URL: https://nagent.ai/privacy-policy. Support contact: **OWNER**. Icon: `plugins/nagent/.claude-plugin/icon.png`. Slug: `nagent` (permanent) | Mostly ready |
| 4. Use cases | Run and follow agents; review approvals; search knowledge; read and update leads. Users need a Nagent workspace on the Team plan or above and an Organisation admin to grant access. Reads and writes data | Ready, **OWNER** to read |
| 5. Company | Company name, website, primary contact for review | **OWNER** |
| 6. Authentication | Required. Method: DCR today; Anthropic-held client or CIMD once ready | **OWNER** |
| 7. Data handling | Own first-party API; no personal health data; no sponsored content | **OWNER** confirm |
| 8. Test and launch | Confirm every tool was run through MCP Inspector or as a custom connector. No credentials in the portal; a reviewer asks for test access by email if the listing goes to Verified review | **OWNER**: run the tools; prepare a demo workspace with an admin login, sample agents, knowledge and leads |
| 9. Compliance | Seven acknowledgments | **OWNER** |
| 10. Review and submit | Final check | **OWNER** |

Draft description:

> Nagent runs your workspace's AI agents for marketing, sales and content.
> With the Nagent connector, Claude can list the agents you may reach, run
> one with a plain-language request, and follow the run until it finishes.
> Each agent works at the autonomy level your workspace set: anything it
> may not do on its own waits for a person in the Nagent Inbox, and Claude
> tells you so. Where your access allows, Claude can also search and add
> Knowledge Hub documents, read and update leads, list and fire triggers,
> and decide approvals as you. Sign-in uses OAuth on nagent.ai; Claude
> reaches only the workspace and permissions you approve.

Screenshots: none. Carousel screenshots are only for MCP Apps, and the
Nagent server returns no UI.

## 4. Plugin submission

| Check | Status |
| --- | --- |
| `.claude-plugin/plugin.json` with `name`, `displayName`, `version`, `description`, `author`, `license` | Ready |
| README of 40 words or more in the plugin folder | Ready: `plugins/nagent/README.md` |
| LICENSE in the plugin folder | Ready |
| Remote MCP server declared with `type: http` and an `https` URL, no credentials in headers | Ready |
| `claude plugin validate` | Passes |
| Name not a reserved word and not confusable with another brand | Ready (`nagent`) |
| Raise `version` in `plugin.json` with each release | **OWNER** at release time |

In the portal choose **Submit new**, then the plugin, enter this repository
and the plugin path `plugins/nagent`, run **Validate**, fix any blocking
finding, then submit. Submit the connector as well and pair the two.
