---
name: nagent-api
description: Use when writing code that calls Nagent agents, approvals, knowledge, leads or triggers through the Nagent REST API (/api/v1), the @nagent/sdk (TypeScript) or nagent (Python) client, or when building a receiver for Nagent webhooks. Covers keys, runs, async runs, sessions, idempotency, errors and webhook signature checks.
---

# Building with the Nagent API

Nagent runs a workspace's AI agents. Code reaches them through the REST API
at `https://nagent.ai/api/v1` with an API key. A run started from code is
the same run as one started in Nagent: the agent works at its autonomy
level, and anything it may not do alone waits for a person.

Full reference: https://nagent.ai/developers. Machine-readable:
https://nagent.ai/api/v1/openapi.json. Endpoint list: `reference.md` next to
this file.

## Rules

- **Never hard-code or log a key.** Read `NAGENT_API_KEY` (and
  `NAGENT_WEBHOOK_SECRET`) from the environment. Keys look like
  `nag_live_…` or `nag_test_…`.
- **Use a test key while building.** It never sends anything and runs agents
  without their tools.
- **Send an `Idempotency-Key` on every run** (a UUID). The same key again
  within 24 hours is refused, so a retry never starts the work twice. Make
  it once per piece of work and reuse it on retries.
- **Long runs: go async.** Send `Prefer: respond-async`, get the run back
  `running`, then poll `GET /runs/{id}` or wait on a webhook.
- **Handle `waiting_for_approval` as a normal outcome**, not an error. The
  run's `waitingOn` lists what a person must decide.
- **Prefer the SDK** when the project is TypeScript/JavaScript or Python.
  Use plain HTTPS otherwise.

## Run shape

`POST /agents/{key}/runs` takes `{ "input": string, "sessionId"?: string,
"metadata"?: { [k]: string } }` and returns a run:

| Field | Meaning |
| --- | --- |
| `id` | Run id |
| `status` | `running`, `completed`, `waiting_for_approval` or `failed` |
| `output` | The agent's answer (only to the key that started the run) |
| `actions` | What the agent did |
| `waitingOn` | Actions held for a person |
| `environment` | `live` or `test` |
| `error` | Why it failed |
| `sessionId`, `metadata` | Echoed back |

Same `sessionId` on later runs = the agent reads the last turns this key ran
in that session (a conversation). `metadata` is your own labels, returned on
the run and its webhook events.

## TypeScript (`npm install @nagent/sdk`, Node 18+, ES module)

```ts
import { Nagent, NagentError } from '@nagent/sdk';
import { randomUUID } from 'node:crypto';

const nagent = new Nagent({ apiKey: process.env.NAGENT_API_KEY! });

const agents = await nagent.agents.list();

const run = await nagent.runs.create(
  'nora',
  { input: 'Research retail-1.example and draft a first email', sessionId: 'crm-4711' },
  { idempotencyKey: randomUUID() },
);

// Long work: accepted at once, then waited on
const accepted = await nagent.runs.create('nora', { input: 'Audit retail-1.example' }, { async: true, idempotencyKey: randomUUID() });
const done = await nagent.runs.wait(accepted.id!);

// Approvals, every page
for await (const a of nagent.approvals.all({ status: 'pending' })) console.log(a.id);
```

Errors throw `NagentError`, carrying the RFC 9457 problem document and the
request id. Other groups: `nagent.webhooks`, `knowledge`, `leads`,
`triggers`; `nagent.agents.markdown(key)` returns an agent's spec as text.

## Python (`pip install nagent`, Python 3.9+)

```python
import os, uuid
from nagent import Nagent, NagentError

nagent = Nagent(api_key=os.environ["NAGENT_API_KEY"])

run = nagent.runs.create("nora", input="Research retail-1.example",
                         session_id="crm-4711", idempotency_key=str(uuid.uuid4()))
print(run["status"], run.get("output"))

accepted = nagent.runs.create("nora", input="Audit retail-1.example", wait=False)
run = nagent.runs.wait(accepted["id"])

for approval in nagent.approvals.iter():
    print(approval["id"])
```

Responses are plain dicts. `NagentError` has `status`, `code` (for example
`agent-not-found`), `problem` and `request_id`. `Nagent(api_key,
base_url="https://nagent.ai", timeout=310.0)`.

## Plain HTTPS

```bash
curl -X POST https://nagent.ai/api/v1/agents/nora/runs \
  -H "Authorization: Bearer $NAGENT_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(uuidgen)" \
  -d '{"input": "Research retail-1.example and draft a first email"}'
```

Add `-H "Prefer: respond-async"` for a long run.

## Errors

Every error is an RFC 9457 problem document (`application/problem+json`)
with `type`, `title`, `status`, `detail` and `instance` (the request id).
The code is the fragment after `#` in `type`.

| Status | Usual cause | Fix |
| --- | --- | --- |
| 401 | No key, or a wrong or revoked key | Check `NAGENT_API_KEY` |
| 403 `scope-missing` | The key was not given the scope | Add the scope on the key |
| 403 `agent-not-allowed` | The key was not given this agent | Add the agent to the key |
| 403 `personal-key-required` | A workspace key tried to decide an approval | Use a personal key |
| 403 `plan` | The plan does not include the API, MCP or webhooks | Team plan or above |
| 404 `agent-not-found` | No agent with that key in the workspace | Check the agent key (`agents.list()`) |
| 409 `duplicate-request` | Same `Idempotency-Key` within 24 hours | Treat as already received |
| 409 `agent-paused` | The agent is paused in the workspace | Resume it in Nagent |
| 429 `rate-limited` | Over the key's requests a minute | Wait for `Retry-After`, then retry |

Log `instance` with any error: it is what Nagent support asks for.

## Webhooks

Add endpoints at https://nagent.ai/admin/settings/api (public https only).
Events: `run.started`, `run.completed`, `run.failed`,
`approval.requested`, `approval.decided`. Body:

```json
{ "id": "<message id>", "type": "run.completed", "createdAt": "...", "data": { } }
```

Events carry the run **without its output**. To get the output, read
`GET /runs/{id}` with the key that started the run.

Signing follows Standard Webhooks: headers `webhook-id`,
`webhook-timestamp`, `webhook-signature`. Always verify the **raw** body
before parsing JSON, reject anything older than 5 minutes, de-duplicate on
`webhook-id`, and answer 2xx fast (do slow work after). Failed deliveries
retry for 24 hours, then the endpoint pauses.

```ts
// Next.js route handler; adapt to the project's framework
import { verifyWebhook } from '@nagent/sdk';

export async function POST(req: Request) {
  const raw = await req.text();
  if (!(await verifyWebhook(process.env.NAGENT_WEBHOOK_SECRET!, req.headers, raw))) {
    return new Response('bad signature', { status: 401 });
  }
  const event = JSON.parse(raw) as { id: string; type: string; data: any };
  // queue the work; de-duplicate on req.headers.get('webhook-id')
  return new Response(null, { status: 204 });
}
```

```python
# Flask; adapt to the project's framework
from nagent import verify_webhook

@app.post("/webhooks/nagent")
def nagent_webhook():
    if not verify_webhook(os.environ["NAGENT_WEBHOOK_SECRET"], request.headers, request.get_data()):
        return "bad signature", 401
    event = request.get_json()
    # queue the work; de-duplicate on request.headers["webhook-id"]
    return "", 204
```

## Keys and scopes

Keys are made on Settings, API & MCP (needs settings permission). API keys
and MCP start on the Team plan.

| Scope | Allows | Plan |
| --- | --- | --- |
| Read agents | List agents, read their setup | Team |
| Run agents | Start runs | Team |
| Read runs | Read status, output, actions | Team |
| Read approvals | List held actions | Business |
| Decide approvals | Approve or reject, as the key's owner. Personal keys only | Business |
| Manage webhooks | Add and remove endpoints | Business |

A workspace key cannot decide approvals; a personal key acts as the person
who made it. Rotating keeps the old key working for 24 hours.
