---
name: mcp-plugin-testing
description: Test MCP servers and plugins across supported clients; create evals and investigate failures. Use when a user wants to check whether models pick the right tools and give useful answers in real clients, compare models or clients, rerun tests after a change, or understand why a test failed. Discover supported clients, models, and agent runtimes with get_eval_options.
metadata:
  owner: manufact
---

# Test an MCP server or plugin in real clients

Evals run a saved conversation against the server's production endpoint in one
or more clients, then a judge scores the result against passing criteria. They
show whether models actually discover and call the right tools, with the right
arguments, and give useful answers. They complement the publishing checklist,
which inspects protocol and metadata only (see the `mcp-plugin-publishing`
skill).

## 1. Pick the server

- Reuse organization and server IDs already established in the conversation.
- Otherwise call `list_servers` without `organizationId` to search every
  accessible organization, and confirm the server with `get_server`. Eval tools
  need both the server's `organizationId` and `serverId`.
- The server needs a healthy production deployment, because evals always target
  the production endpoint. If it has none, use the `mcp-plugin-deploy` skill
  first. To test a branch, promote or merge it first; preview URLs are not
  eval targets.
- If the server requires OAuth, complete `authenticate_deployment` first as
  described in `mcp-plugin-publishing`, so clients can connect.

## 2. Discover clients and models

Call `get_eval_options` before choosing clients. Use only what it returns:

- **Inspector**: the Manufact chat client. Choose any number of OpenRouter
  models with `list_eval_models` (filter with `query` or `provider`). Each model
  runs as its own execution. Omitting clients uses Inspector with the default
  model.
- **Web clients** (`webClients`): ChatGPT and Claude on the web. Each runs once
  per suite with no model choice and does not enforce `disallowedTools`.
- **Runtimes**: agent harnesses such as Codex, Claude Code, or Cursor. Pick
  only runtimes with `available: true`, choose a model from that runtime's
  `models`, and tell the user about any `limitations` or the `reason` it is
  unavailable.

Other MCP clients can still connect to the server's MCP URL, but they can't be
tested here unless `get_eval_options` lists them. Say so instead of implying
coverage.

## 3. Create or update suites

Check `list_eval_suites` first and reuse a matching suite with
`update_eval_suite` rather than creating a duplicate. Read one with
`get_eval_suite` when you need its full configuration.

Create a suite with `create_eval_suite` for each behavior worth checking:

- **`flow`**: the conversation in order. Use realistic user wording, not the
  tool's name. Add a `tool_call` step after a user step when the model must
  call a specific tool; match only the arguments that matter (`"any"`,
  `contains`, or `pattern`).
- **`rubric`**: concrete, observable passing criteria the judge can check.
- **`threshold`**: the minimum judge score to pass; the default is 0.7.
- **`clients`**: the clients and models chosen in step 2.
- **`disallowedTools`**: tools the model must not call, such as destructive
  tools in a read-only scenario.

Cover the main jobs users bring to the server, plus at least one case where the
model should decline or ask for clarification. If you can read the server's
source, derive the cases from its actual tools.

## 4. Run the suites

1. If the user asked to run tests, call `run_eval_suites` with up to ten
   `suiteIds` without asking again. If you are starting runs on your own
   initiative, say which suites and clients will run first: each execution
   uses plan resources.
2. Each suite starts its own background run. Read each receipt's `outcome`; a
   suite that was `not_started` or has an `error` did not run.
3. If the start response was lost or uncertain, check `list_eval_runs` before
   starting again. `cancel_eval_run` stops a run that is no longer needed.

## 5. Read the results

1. Call `get_eval_report` with the returned `runIds`. Runs take a few minutes.
   While a run is still in progress, tell the user and check again when they
   ask; don't poll in a tight loop.
2. Report per client and model: pass or fail, score, and the tool calls made.
3. Use `list_eval_runs` to compare with earlier runs of the same suite.

## 6. Investigate failures

For each failed execution, walk through the judge findings and the tool calls:

| Symptom | Likely cause and fix |
| --- | --- |
| The expected tool was never called | The tool's title or description does not match how users ask. Improve the description, or check that the tool is exposed to that client. |
| The wrong tool was called | Two tools overlap. Sharpen both descriptions so each states when to use it. |
| Right tool, wrong arguments | Parameter descriptions are vague, or the schema allows ambiguous values. Describe each parameter and its format. |
| The tool errored | Read `get_deployment_runtime_logs` or the `mcp-plugin-analytics` skill's issue tools for the server-side error. |
| Tools worked but the answer failed the rubric | The output lacks what the answer needs, or the rubric is stricter than intended. Fix the output or the rubric, not both at once. |
| Only one client fails | Check that client's `limitations` and whether it supports the feature in use, such as widgets or authentication. |
| The run failed before executing | Connection or authentication failed. Fix access, then rerun. |

Fix the cause in the server's code, push, and let the deployment finish. Rerun
only the affected suites, once something has changed. Report what passed, what
failed and why, and what remains untested, including clients that
`get_eval_options` did not offer.
