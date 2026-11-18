---
description: Run one of your Nagent agents and show the result
argument-hint: "<agent key> <what the agent should do>"
---

Run a Nagent agent over MCP.

Arguments: `$ARGUMENTS`. The first word is the agent key; the rest is the
ask, passed to the agent word for word.

1. If no `nagent_*` MCP tools are available, say the Nagent MCP server is not
   connected and suggest `/nagent:setup`. Stop.
2. If there are no arguments, or the first word is not an agent key, call
   `nagent_list_agents`, show the keys and names, and ask which one. Stop.
3. Call the tool `<agent key>_run` with `input` set to the ask.
4. Show the outcome plainly:
   - `completed`: the output.
   - `waiting_for_approval`: what it is waiting on, and that a person must
     decide it in the Nagent Inbox (https://nagent.ai). Do not decide it
     yourself unless the developer asks you to.
   - `running`: call `nagent_wait_run` with the run id, then show the result.
   - `failed`: the error, and the run id so they can look it up.
