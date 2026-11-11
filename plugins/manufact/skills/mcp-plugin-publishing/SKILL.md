---
name: mcp-plugin-publishing
description: Check publishing readiness for MCP servers and plugins; prepare supported directory submissions. Use when a user wants to know whether an MCP server or MCP app is ready for supported AI agent directories, run or fix the Manufact publishing checklist, or package an Agent Plugin with listing metadata, review cases, and a demo recording. For standalone behavior testing, use mcp-plugin-testing.
metadata:
  owner: manufact
---

# Get an MCP server or plugin ready to publish

App directories such as the ChatGPT app directory and Claude's connector
directory reject servers for protocol errors, missing tool metadata, broken
widgets, and policy problems. Manufact finds these before a reviewer does.

"Ready" means:

- the server is deployed and healthy,
- the publishing checklist has no blocking failures for the target platform,
- the server behaves correctly in real clients, and
- the plugin package and required listing and review materials are complete.

For server readiness, review the code and work through the checks below in order.
The live checklist result is authoritative; the reference files explain checks
but can lag behind it. Don't quote plan names, prices, or directory rules from
memory.
For a packaging or ZIP-only request, start at step 8, reuse available readiness
evidence, and
report unverified behavior. Package preparation can proceed without a checklist
run or submission-portal access.

## 1. Pick the server and target

1. Reuse a server ID already established in the conversation. Otherwise find
   the server with `list_servers`, omitting `organizationId` to search every
   accessible organization, and confirm it with `get_server`. The server must
   have a live deployment. If it doesn't, use the `mcp-plugin-deploy` skill
   first.
2. Resolve which platforms the user is targeting. The checklist reports
   readiness for the platforms it returns, currently ChatGPT and Claude; the
   answer decides which failures matter. Other clients, such as Cursor, Grok,
   or Muse, can connect to the server's MCP URL, but this checklist does not
   assess a directory for them.
3. Check whether a run already exists with `list_checklist_runs`. If a recent
   run matches the current deployment, read it with `get_checklist_run` instead
   of starting a new one.

## 2. Review the code first

If you can read the server's source, check it against the common blocking
failures before running anything. See
[references/checklist-checks.md](references/checklist-checks.md) for the full
list.

- **Every tool has a top-level `title` and a description.**
- **Every tool sets boolean `readOnlyHint`, `destructiveHint`, and
  `openWorldHint`.** The values must match what the tool actually does.
- **Tool names are unique, at most 64 characters, and use only letters, digits,
  `_`, `-`, or `.`.**
- **Input schemas are objects.** Declared output schemas are valid.
- **For MCP apps with widgets:**
  - Each widget resource uses a `ui://` URI with the MIME type
    `text/html;profile=mcp-app`.
  - It declares one exact HTTPS domain.
  - Its CSP lists every external origin it loads or connects to.
- **Listings don't overpromise.** Descriptions and behavior avoid policy
  problems: misleading claims, collecting sensitive data without need, and
  unexpected destructive actions.

Fix what you find, commit, push, and let the deployment finish before
continuing.

## 3. Authenticate protected servers

If the server requires OAuth, the checklist needs credentials to connect.

1. Call `authenticate_deployment` with `serverId`. Pass `branch` only for a
   preview. Clients that support it open the consent page directly and get
   the session's status back. Otherwise give the user the returned
   `authenticationUrl` as a clickable link.
2. If the result is still `pending`, wait until the user says they're done,
   then call it once with the returned `sessionId`. Don't poll.
3. Only `authenticated` means success. `pending` means the user hasn't
   finished. `expired` or `failed` means you need a new link.
4. Never ask the user for OAuth credentials, tokens, or passwords.

## 4. Run the publishing checklist

1. Each call starts a new run. Reuse an existing run when it matches the current
   deployment.
2. Call `run_publishing_checklist` with `serverId`. Omit `branch` to check
   production. When the user asked for a run, start it without asking again.
3. **Read the run with `get_checklist_run`** and the returned `runId`. Runs take
   several minutes. `completed` and `failed` are final; `pending` and `running`
   mean the run is still in progress.
   - If the start response was lost or uncertain, look at
     `list_checklist_runs` before starting another run.
4. **Read the final result:**
   - `isReadyForChatgpt` and `isReadyForClaudeai`: a platform is ready when no
     check with severity `error` fails for it. Warnings and info never block.
   - `score`: a 0–100 quality signal that matches the dashboard. It is not the
     readiness verdict.
   - A `failed` run means the checklist itself couldn't finish, for example
     because the server was unreachable or authentication failed. It says
     nothing about the server's quality. Fix the cause before rerunning.
5. **Report back grouped by priority:**
   1. Blocking errors for the user's target platforms.
   2. Warnings worth fixing.
   3. Everything else, briefly.

   Quote each failing check's `message` and `hint` when available. If `message`,
   `details`, or `hint` is `null`, explain the check from the reference file
   without inventing missing findings.

## 5. Fix and verify

- **Fix issues in the server's code**, using the hints and
  [references/checklist-checks.md](references/checklist-checks.md). Then push and
  let the deployment finish. With a connected repository, the dashboard can also
  open an autofix pull request.
- **Rerun the checklist once** after the fixes are live. Don't rerun until
  something has changed.
- Automatic runs after each deployment are available through
  `update_server_checklist_settings`. Enable them only
  when the user asks.

## 6. Test real behavior

Checklist checks look at metadata. To check whether models actually pick the
right tools and give useful answers in real clients, follow the
`mcp-plugin-testing` skill. Passing evals are useful evidence for the review
cases in step 9.

## 7. Let people try it

A public chat page lets anyone try the server in a browser, with no MCP client
needed.

- Check the current state with `get_public_chat_configuration`.
- Enable it with `configure_public_chat`: `enabled: true`, a title, and up to
  eight suggested questions that show off the main tools.

## 8. Prepare the Agent Plugin package and listing

Use the [Agent Plugins specification](https://agent-plugins.org/specification)
and [references/plugin-packaging.md](references/plugin-packaging.md). Read the
reference before writing manifests or preparing review materials.

No Manufact tool generates or stores a submission pack. Build the package from
the source with the steps below, and complete upload, review, and publication
in the directory's own submission portal, such as the
[OpenAI plugin submission portal](https://developers.openai.com/plugins/deploy/submission).

1. Inspect the actual source, deployed MCP endpoint, existing manifests, skills,
   and supplied listing materials. Preserve an existing plugin's identity,
   hosting, authentication, and audience.
2. Build a portable package with root `plugin.json`, root `mcp.json` for MCP
   connections, `skills/<name>/SKILL.md` for skills, and referenced assets.
   Use the matching Agent Plugins schema declarations. Put OpenAI listing,
   review, and publication fields under `extensions.com.openai`.
3. Draft listing copy from supported behavior. Collect only missing publisher,
   country targeting and policy facts. Help complete the icon and
   all four public listing URLs; verify their content and accessibility.
4. Prepare public uploads in a separate copy. Use verified MCP endpoints and
   exclude private app bindings and credentials. Preserve the original source
   and any bindings the submission portal generates in saved releases.

## 9. Prepare review materials

- For each app needing initial public review, draft five positive and three
  negative cases from its actual tools and workflows. Give each a reproducible
  setup and observable pass condition. Record which cases passed, failed, were
  blocked, or were not run; a draft case is not execution evidence.
- Help record a real demo of the packaged version in the intended host. Rehearse
  the prompts, capture actual tool results, inspect playback, and verify the
  reviewer-accessible recording URL before writing it into the manifest.
  Screenshots alone or a walkthrough script do not complete the recording.
- Keep reviewer credentials and login instructions in secure dashboard fields,
  outside the ZIP. Use a dedicated test account with the required access.
- Write review and publication metadata into the package using the reference's
  field mappings. Skills-only plugins need no MCP cases, demo, or credentials.
  Existing reviewed dependencies need an eligible version.

## 10. Validate and hand off

1. Build a ZIP containing one plugin at the root or under one enclosing folder.
   Parse the final manifests, inspect all entries including hidden compatibility
   files, validate referenced assets and skills, and check listing and review
   metadata. Rebuild and inspect again whenever the source changes.
2. Return the archive with what is complete and the specific gaps. Report package
   readiness separately from remaining portal requirements and unverified runtime
   behavior. A valid package or clean checklist does not guarantee approval.
3. For ZIP-only requests, deliver the ZIP without creating or uploading another
   plugin. Upload only when requested. Final portal setup consists of a draft
   upload, connecting/authenticating the actual server, and testing the saved
   version. Verify imported metadata before any authorized submission.
4. Draft upload, submission for review, and publishing an approved release are
   distinct actions. Report only the stage actually completed. Have the
   authorized developer complete legal and policy attestations.
