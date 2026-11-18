---
name: nagent-setup
description: "Set up Nagent: connect this agent to the Nagent MCP server with OAuth, list the Nagent agents it can reach, and optionally wire the Nagent SDK, an example and a webhook receiver into the current project. Use when the developer asks to set up, connect or install Nagent."
---

<!-- Generated from shared/nagent-setup.md by scripts/sync-shared.py. Edit the source, not this file. -->

# Set up Nagent

Walk the developer through setting up Nagent. Keep each message short. Do one
step, check it worked, then move on. Never print, store or commit a real API
key or webhook secret: they go in the developer's own secrets store or an
ignored `.env` file, which the developer fills in themselves.

If the developer named where to start, follow it:
- `mcp` or nothing: start at step 1.
- `app`: skip to step 3.
- `webhooks`: skip to step 4.

## Step 1. Connect over MCP

Nagent is an MCP server at `https://nagent.ai/api/mcp/nagent`. It signs in
with OAuth in the browser, so no key is needed.

1. Check whether the Nagent MCP tools are available to you (for example
   `nagent_list_agents`). If they are, call `nagent_list_agents` and go to
   step 2.
2. If not, tell the developer how to connect in the app they are using, then
   ask them to say when it is done and call `nagent_list_agents`:
   - **Codex CLI**: `codex mcp add nagent --url https://nagent.ai/api/mcp/nagent`
     if the server is not listed in `codex mcp list`, then
     `codex mcp login nagent`. Start a new session afterwards.
   - **Codex in the ChatGPT desktop app**: open **Plugins**, find Nagent and
     connect it.
   - **Cursor**: open **Customize** (or Cursor Settings, MCP), find the Nagent
     server and choose **Connect**. If it is missing, add it with the
     "Add to Cursor" link in the Nagent setup guide:
     https://github.com/PWIN-Technologies-Pvt-Ltd/nagent-claude-plugin/tree/main/cursor
   - **Claude Code**: run `/mcp`, pick `nagent` and choose **Authenticate**.

   A browser opens on nagent.ai: they sign in, pick the workspace and allow
   access.
3. If sign-in fails or the workspace has no API access, explain the likely
   cause in one line:
   - API keys and MCP start on the **Team** plan.
   - Only people who may change settings (an Organisation admin) can grant
     access.
   - A workspace on its own domain (a private-label copy) uses its own host:
     the same steps, with `https://<host>/api/mcp/nagent` as the server URL.

## Step 2. Show what they can do from here

From the `nagent_list_agents` result, show a short table: agent key, name,
autonomy level. With more than 15 agents, do not list them all: give the
count by autonomy level and a table of at most 10 (switched-on agents with
the highest autonomy first), and say they can name any other. If the result
is too large to read whole, read it in parts or filter it, and say how many
agents there are. Then say, in two or three lines:

- Each agent is a tool named `<agent key>_run`. Ask in plain words, for
  example "ask nora to research retail-1.example", or use `nagent-run`.
- A run works at the agent's autonomy level. Anything it may not do on its
  own waits for a person in the Nagent Inbox, and the result says so.
- `nagent_get_run` and `nagent_wait_run` follow a run; other `nagent_*` tools
  (knowledge, leads, triggers, approvals) appear only when the access allows.

Then ask whether they also want to call Nagent from their own code. If not,
or if you cannot read or change files in this session, stop here and point
to https://nagent.ai/developers.

## Step 3. Call Nagent from this project

1. Look at the project to pick the language: `package.json` means
   TypeScript or JavaScript (`npm install @nagent/sdk`, Node 18+, ES module);
   `pyproject.toml`, `requirements.txt` or `setup.py` means Python
   (`pip install nagent`, Python 3.9+). Anything else: plain HTTPS, as the
   `nagent-api` guidance shows. Say which you picked and why, in one line.
2. Tell them to create a key at https://nagent.ai/admin/settings/api:
   **Create key**, Environment **Test** while building (a test key never
   sends anything and runs agents without tools), only the scopes they need
   (usually Read agents, Run agents, Read runs), and **Only these** agents if
   the code needs just one. The key is shown once.
3. Add `NAGENT_API_KEY=` (empty) to `.env.example`, creating it if needed.
   Make sure `.env` is listed in `.gitignore`. Tell them to put the real key
   in `.env` or their secrets store. Do not write the key anywhere yourself.
4. Install the SDK with the project's own package manager (look for
   `pnpm-lock.yaml`, `yarn.lock`, `bun.lockb`, `uv.lock`, `poetry.lock`).
5. Write one small, runnable example in the project's style, following the
   `nagent-api` guidance: list agents, then start one run with an
   Idempotency-Key and print its status and output. Put it where examples
   or scripts already live in the project, or in `scripts/` if there is no
   such place.
6. Once they say the key is in place, run the example. On a `401`, the key
   is missing or wrong; on `403 scope-missing` or `403 agent-not-allowed`,
   the key lacks that scope or agent; on `404 agent-not-found`, no agent has
   that key (list them first).

## Step 4. Webhooks (optional)

Ask whether their system should hear back from Nagent (run finished, run
failed, an approval waiting or decided). If yes:

1. Add a receiver route in their web framework that reads the **raw** body,
   verifies it with `verifyWebhook` (TypeScript) or `verify_webhook`
   (Python) and `NAGENT_WEBHOOK_SECRET`, answers 2xx quickly, and handles
   `run.completed`, `run.failed` and `approval.requested`. Follow the
   `nagent-api` guidance.
2. Add `NAGENT_WEBHOOK_SECRET=` (empty) to `.env.example`.
3. Tell them to add the endpoint under **Webhooks** on the same settings
   page (public https URL only), copy the signing secret it shows once into
   `.env`, and press **Send test event**.

## Finish

End with a short list of what was set up and the one thing left for them to
do, if any. Point to https://nagent.ai/developers for the full reference.
