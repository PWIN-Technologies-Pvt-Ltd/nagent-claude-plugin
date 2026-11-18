# OpenAI plugin directory: submission packet for Nagent

The OpenAI plugin directory is shared by ChatGPT and Codex: one approved
plugin is listed in both. This packet lists everything the submission needs.
Items marked **OWNER** need a decision or material from the Nagent owner.
Items marked **SERVER** need a change in the nagent-website repository
before review. Everything else is ready in this repository.

Sources: [Upload and submit your plugin](https://developers.openai.com/plugins/deploy/submission),
[Remote MCP server review requirements](https://developers.openai.com/plugins/deploy/app-review),
[Plugin guidelines](https://developers.openai.com/plugins/plugin-guidelines),
[Authentication](https://developers.openai.com/plugins/build/auth),
[Package your plugin](https://developers.openai.com/plugins/build/plugins).

## 1. What we submit

- **Package**: a ZIP of `codex/plugins/nagent/` (`plugin.json`, `mcp.json`,
  `skills/`, `assets/`, `LICENSE`). No hooks and no `.app.json`, which the
  portal rejects.
- **Type**: **With MCP**. One remote server, `nagent`,
  `https://nagent.ai/api/mcp/nagent`, Streamable HTTP, OAuth with dynamic
  client registration (declared in `mcp.json`).
- **Portal**: https://platform.openai.com/plugins, **Upload new or existing
  plugin**.

Make the ZIP from inside the folder so `plugin.json` is at its root:

```bash
python3 scripts/sync-shared.py --check
cd codex/plugins/nagent && zip -r ../../../nagent-openai-plugin.zip . -x '.*'
```

## 2. Before you start

| Item | Status |
| --- | --- |
| OpenAI Platform organisation and a project with global data residency (EU residency projects cannot submit MCP plugins) | **OWNER** |
| Business verification for the publishing name in [organisation settings](https://platform.openai.com/settings/organization/general). Decide the publisher name shown in the directory: "Nagent" or the registered company name | **OWNER** |
| Apps Management Write permission for whoever submits (organisation owners have it) | **OWNER** |

## 3. Listing (from `plugin.json`)

| Field | Value | Status |
| --- | --- | --- |
| Plugin id | `nagent` | Ready, permanent |
| Display name | Nagent | Ready |
| Short description | Run your Nagent agents and build with the Nagent API | Ready |
| Long description | In `plugin.json` (`longDescription`) | Ready, **OWNER** to read |
| Developer name | Nagent | **OWNER** confirm against the verified identity |
| Category | Productivity | **OWNER** confirm; pick from the portal's list |
| Capabilities | Read, Write | Ready |
| Website | https://nagent.ai/developers | Ready |
| Support URL | https://nagent.ai/contact-us | **OWNER** confirm it is the support channel |
| Privacy policy | https://nagent.ai/privacy-policy | **OWNER** confirm it covers categories of personal data, purposes, recipients, retention and user controls, including data that passes through the MCP tools (leads, knowledge, run output) |
| Terms of service | https://nagent.ai/terms-and-conditions | **OWNER** confirm current terms |
| Starter prompts | 3 in `defaultPrompt` | Ready |
| Icon and logo | `assets/icon.png`, 1024 x 1024 PNG, the official Nagent mark | Ready. Optional dark variants and brand colour: **OWNER** |
| Countries | Not set | **OWNER** (add `publication.countries` or set in the portal) |
| Translations | None | Optional |

Guideline checks done: the name has no "MCP" or "Plugin" suffix; the
descriptions make no price, trial or comparison claims; nothing implies
OpenAI endorsement.

## 4. Authentication

| Item | Status |
| --- | --- |
| Protected resource metadata at `/.well-known/oauth-protected-resource/api/mcp/nagent`, `resource` equals the server URL, `401` carries `WWW-Authenticate: Bearer resource_metadata=...` | Ready (checked 18 Nov 2026) |
| Authorization server metadata with `registration_endpoint`, PKCE `S256`, `token_endpoint_auth_methods_supported` including `none` | Ready |
| RFC 9207 issuer identification: `authorization_response_iss_parameter_supported: true` and `iss` on every authorization response. ChatGPT then uses the stable redirect `https://chatgpt.com/connector_platform_oauth_redirect` | Ready in code; confirm in the first live sign-in that `iss` is exactly `https://nagent.ai` |
| Redirect URIs: Nagent accepts any `https` redirect registered through DCR, so both ChatGPT redirects (`https://chatgpt.com/connector_platform_oauth_redirect` and `https://chatgpt.com/connector/oauth/{callback_id}`) work | Ready |
| `openid` and `email` scopes, OpenID discovery, and a UserInfo endpoint returning `email` and `email_verified: true` (OpenAI's submission guidance asks for these with OAuth) | **SERVER** |
| Domain verification: serve the portal's token as plain text at `https://nagent.ai/.well-known/openai-apps-challenge` (currently 404) | **SERVER** route, **OWNER** pastes the token |
| Least-privilege consent: today a client is offered all 17 scopes. Add `scope="agents:read agents:run runs:read"` to the `401` challenge or trim the default request, so the consent screen asks only for what the plugin needs | **SERVER** (recommended) |
| Optional: Client ID Metadata Documents (`client_id_metadata_document_supported: true`), which ChatGPT prefers over DCR | **SERVER** (optional) |

## 5. Tools and annotations

OpenAI requires explicit `readOnlyHint`, `destructiveHint` and
`openWorldHint` booleans on every tool. The Nagent server sends none today,
and only `<agent>_run` has a `title`. Without `readOnlyHint`, ChatGPT treats
every tool as a write and asks for confirmation each time. **SERVER**: add
these values (proposed; **OWNER** to confirm the ones marked).

| Tool | Title | readOnly | destructive | openWorld | Scope |
| --- | --- | --- | --- | --- | --- |
| `<agent key>_run` | Run `<agent name>` | false | true (**OWNER**: an agent at a high autonomy level can send email or publish without a person) | true | agents:run |
| `nagent_list_agents` | List agents | true | false | false | agents:read |
| `nagent_get_run` | Get a run | true | false | false | runs:read |
| `nagent_wait_run` | Wait for a run | true | false | false | runs:read |
| `nagent_read_smriti` | Read agent memory | true | false | false | agents:read |
| `nagent_list_proposals` | List SEO and AEO proposals | true | false | false | agents:read |
| `nagent_decide_approval` | Decide an approval | false | true | true | approvals:decide |
| `nagent_search_knowledge` | Search knowledge | true (**OWNER**: semantic mode spends credits) | false | false | knowledge:read |
| `nagent_read_knowledge` | Read a knowledge document | true | false | false | knowledge:read |
| `nagent_add_knowledge` | Add a knowledge document | false | false | true (reads a public URL when given one) | knowledge:write |
| `nagent_list_leads` | List leads | true | false | false | leads:read |
| `nagent_get_lead` | Get a lead | true | false | false | leads:read |
| `nagent_create_lead` | Create a lead | false | false | false | leads:write |
| `nagent_update_lead` | Update a lead | false | true (overwrites fields) | false | leads:write |
| `nagent_list_triggers` | List triggers | true | false | false | ops:triggers |
| `nagent_fire_trigger` | Fire a trigger | false | true | true | ops:triggers |
| `nagent_subscribe_event` | Create an event trigger | false | false | false | ops:triggers |

Other tool findings to fix before review (**SERVER**):

- **Per-agent tool names.** Each workspace gets different `<agent key>_run`
  tools. OpenAI stores the tool list scanned from the review account and
  holds any new tool until it passes automated checks, so other users'
  agent tools may never appear. Add one stable tool, for example
  `nagent_run_agent` with `agentKey` and `input`, and keep the per-agent
  tools as a convenience.
- **Response size.** `nagent_list_agents` returns about 153,000 characters
  for a big workspace. Return a compact list (key, name, autonomy, enabled)
  with paging, and leave descriptions to a separate call.
- **Response minimisation.** Return only what the request needs; drop
  internal ids and request ids from tool text unless the next call needs
  them (run ids are needed).

## 6. Review information

| Item | Status |
| --- | --- |
| Test account: a dedicated demo workspace on Team or Business, an Organisation admin login (only admins can grant access), sign-in with email and password and **no** MFA, email code or magic link, sample agents, knowledge and leads. Credentials go in the portal, never in the package | **OWNER** |
| Five positive test cases (draft below; run each with the test account and record the expected result) | **OWNER** to run |
| Three negative test cases (draft below) | **OWNER** to run |
| Video walkthrough URL showing the test cases | **OWNER** |
| Release notes: "First release: run Nagent agents, follow runs and approvals, and set up the Nagent API, SDKs and webhooks." | Ready |
| Screenshots: none required for a plugin without UI; the video covers it | Ready |

Draft positive cases:

1. "List my Nagent agents." Tools: `nagent_list_agents`. Expected: the demo
   agents with key, name and autonomy level.
2. "Ask `<demo agent>` to research retail-1.example." Tools:
   `<demo agent>_run`. Expected: a completed run with the research, or
   `waiting_for_approval` naming the held action.
3. "Is run `<id>` finished?" Tools: `nagent_get_run` or `nagent_wait_run`.
   Expected: the run's status and output.
4. "Find our knowledge documents about onboarding." Tools:
   `nagent_search_knowledge`. Expected: matching demo documents.
5. "Show the newest leads." Tools: `nagent_list_leads`. Expected: the demo
   leads, newest first.

Draft negative cases:

1. "Transfer 100 dollars to my supplier." Expected: no tool call; Nagent
   does not move money.
2. "Delete every lead in my workspace." Expected: no tool call; there is no
   delete tool, and ChatGPT says so.
3. "Show me another company's Nagent agents." Expected: no tool call
   outside the connected workspace; explain that access is limited to the
   connected account.

## 7. Policy notes

- **Commerce**: the plugin sells nothing. The setup skill mentions that API
  access starts on the Team plan, which the guidelines allow as plan
  information; it does not link to checkout.
- **Credentials**: the setup skill tells developers to make a key on
  nagent.ai and put it in their own `.env`; it never asks for a key in chat.
- **Approvals**: the skills tell the model not to decide approvals unless
  the user asks.

## 8. Submit

1. Upload the ZIP, wait for the metadata and skill checks, fix findings.
2. **MCPs**, **Connect**: complete domain verification, connect with the
   test account, wait for **Scan Tools**, fix findings.
3. **Review information**: enter the test account and test cases.
4. **Submit for review** and accept the attestations.
5. After approval, **Publish plugin**.

Later server changes are picked up by the daily scan; changes to the
package (skills, listing) need a new ZIP with a higher `version`.
