# Nagent REST API v1: endpoints

Base URL `https://nagent.ai/api/v1`. Auth `Authorization: Bearer $NAGENT_API_KEY`.
Spec: https://nagent.ai/api/v1/openapi.json. Each SDK method maps to one
`operationId`.

| Method | Path | operationId | TypeScript | Python |
| --- | --- | --- | --- | --- |
| GET | /agents | listAgents | `agents.list()` | `agents.list()` |
| GET | /agents/{key} | getAgent | `agents.get(key)` | `agents.get(key)` |
| GET | /agents/{key}/agent.md | getAgentMarkdown | `agents.markdown(key)` | `agents.markdown(key)` |
| POST | /agents/{key}/runs | createRun | `runs.create(key, body, opts)` | `runs.create(key, input, ...)` |
| GET | /runs/{id} | getRun | `runs.get(id)` | `runs.get(id)` |
| GET | /runs/{id}/events | getRunEvents (SSE) | `runs.events(id)` | `runs.events(id)` |
| GET | /approvals | listApprovals | `approvals.list()` / `.all()` | `approvals.list()` / `.iter()` |
| POST | /approvals/{id}/decide | decideApproval | `approvals.decide(id, {decision})` | `approvals.decide(id, decision)` |
| GET | /webhooks | listWebhooks | `webhooks.list()` | `webhooks.list()` |
| POST | /webhooks | createWebhook | `webhooks.create(...)` | `webhooks.create(url, events)` |
| DELETE | /webhooks/{id} | deleteWebhook | `webhooks.delete(id)` | `webhooks.delete(id)` |
| GET | /knowledge | listKnowledge | `knowledge.list()` | `knowledge.list(q, limit)` |
| POST | /knowledge | createKnowledge | `knowledge.create(...)` | `knowledge.create(title, text, url)` |
| GET | /knowledge/{id} | getKnowledge | `knowledge.get(id)` | `knowledge.get(id)` |
| GET | /leads | listLeads | `leads.list()` | `leads.list(...)` |
| POST | /leads | createLead | `leads.create(...)` | `leads.create(**fields)` |
| GET | /leads/{id} | getLead | `leads.get(id)` | `leads.get(id)` |
| PATCH | /leads/{id} | updateLead | `leads.update(id, ...)` | `leads.update(id, **fields)` |
| GET | /triggers | listTriggers | `triggers.list()` | `triggers.list(agent)` |
| POST | /triggers/{id}/fire | fireTrigger | `triggers.fire(id)` | `triggers.fire(id)` |

Other operations in the spec (tool connectors, conversions, Convexa, Assist)
are for specific products; read the spec before using them.

Lists page with `limit` and `cursor`; a page carries `hasMore` and
`nextCursor`. A `decideApproval` body is `{ "decision": "approve" | "reject",
"note"?: string }` and needs a personal key.
