---
name: manufact-getting-started
description: Get started with Manufact Cloud to build, deploy, test, and monitor MCP servers and plugins. Use right after a user connects Manufact, or when they ask what Manufact does or where to start. Covers the account model, a quick-start template deploy that needs no GitHub, and routing to deployment, testing, analytics, or publishing for the most popular AI Agents.
metadata:
  owner: manufact
---

# Get started with Manufact Cloud

Manufact Cloud builds, hosts, tests, and monitors MCP servers and MCP apps,
which users install as plugins or connectors in their clients. It is built by
the team behind the open-source mcp-use SDKs, but it hosts servers built with
any framework. A deployed server gets a live MCP URL that any MCP client can
connect to, including ChatGPT, Claude, Cursor, Grok, and Muse.

Not every feature covers every client. Read support from the tools rather than
from this list: `get_eval_options` returns the clients and runtimes that evals
can run in, and the publishing checklist reports readiness only for the
platforms it returns.

If Manufact tools require sign-in, complete Cursor's OAuth connection flow for
the Manufact MCP server. Never ask users for passwords, API keys, or tokens.

## How Manufact Cloud is organized

```text
User
└── Organizations        (a user belongs to one or more)
    └── Servers          (each server is one MCP server)
        └── Deployments  (each server has many; one serves production)
```

- **User**: the signed-in account. It can belong to several organizations.
- **Organization**: owns servers, GitHub connections, and members.
  New accounts get a Primary organization automatically.
- **Server**: one MCP server. It has a source repository, build settings,
  environment variables, domains, analytics, checklist runs, eval suites,
  and an optional public chat page.
- **Deployment**: one build of the server from a branch and commit.
  - The production branch (usually `main`) deploys on every push and serves
    `https://<slug>.run.mcp-use.com/mcp`.
  - Other branches get preview URLs.
  - A newer deployment can fail while an older one keeps serving traffic.

Tools follow the same hierarchy. Organization IDs scope servers, server IDs
scope deployments, and so on down. Use only IDs that tools return or that the
user gives you. Never guess an ID, and reuse IDs already established in the
conversation instead of looking them up again.

## First conversation

1. **Introduce Manufact briefly.** One or two sentences is enough, unless the user
   asks for more.
2. **Look at the account before suggesting anything.** Call `list_servers`
   without `organizationId`; it lists servers across every accessible
   organization, grouped by organization. Call `list_organizations` only when
   you need organization names or an organization with no servers.
3. **Summarize what you found**, in plain language: which organizations, how
   many servers, and each server's name and latest deployment status. Call
   `get_server` only for a server the user wants to look at more closely.
4. **Offer the next steps that fit their state**, as the table below shows. Offer
   two or three concrete options, not the whole tool list.

| User's state | Suggested next step |
| --- | --- |
| No servers, wants to try Manufact quickly | Deploy a starter with `create_server_from_template`. It needs no GitHub connection and starts the first deployment. |
| No servers, has an MCP server repo on GitHub | Deploy it. Follow the `mcp-plugin-deploy` skill. |
| Server with a failed latest deployment | Debug it with the recovery steps in the `mcp-plugin-deploy` skill. |
| Healthy server, wants to know whether it works in real clients | Follow the `mcp-plugin-testing` skill to create and run evals. |
| Healthy server, not yet published | Check publishing readiness. Follow the `mcp-plugin-publishing` skill. |
| Healthy server, wants others to try it | Enable a public chat page (`configure_public_chat`). |
| Server, wants to understand usage or errors | Follow the `mcp-plugin-analytics` skill to inspect scoped metrics and evidence. |
| Server hosted elsewhere | Connect it in the dashboard (see below), then run the checklist. |

## Quick start without GitHub

`create_server_from_template` is the fastest path to a live MCP URL. It needs
only `organizationId`; when the user has a single organization, use it without
asking. It deploys the `mcp-apps` starter unless the user picks `mcp-server` or
`blank`, keeps the source in a private Manufact-managed repository, and starts
the first deployment. Follow the deployment as described in
`mcp-plugin-deploy`. Users who already have code on GitHub should deploy that
repository instead.

## What each area covers

| Area | Main tools | Use for |
| --- | --- | --- |
| Organizations and team | `list_organizations`, `create_organization`, `list_team_members`, `send_team_invite` | Choose a workspace and invite teammates. |
| GitHub | `connect_github`, `list_github_installations`, `list_github_repositories`, `connect_github_repo_to_server` | Give Manufact access to repositories. |
| Servers and deployments | `create_server_from_template`, `create_server`, `deploy`, `get_deployment`, `list_deployments`, `promote_deployment` | Ship code and follow builds. |
| Configuration | `get_server_configuration`, `update_server_deployment_settings`, `create_environment_variable` | Change build settings and secrets. |
| Domains | `get_server_domains`, `update_server_subdomain`, `add_server_custom_domain` | Change the server's URLs. |
| Testing | `get_eval_options`, `create_eval_suite`, `run_eval_suites`, `get_eval_report` | Test behavior in real clients; follow `mcp-plugin-testing`. |
| Observability | `get_server_analytics_summary`, `get_server_analytics_catalog`, `get_deployment_runtime_logs` | See usage and errors; follow `mcp-plugin-analytics` for deeper questions. |
| Publishing | `run_publishing_checklist`, `configure_public_chat` | Check readiness; follow `mcp-plugin-publishing` to package a plugin for supported directories. |

## What happens outside this MCP server

Send the user to the dashboard at https://manufact.com, or to the CLI, for these:

- **Connecting a server hosted elsewhere** (by URL, optionally proxied through
  Manufact). No tool here creates external servers.
- **Deploying the user's own code without GitHub.** Run `npx @mcp-use/cli login`,
  then `npx @mcp-use/cli deploy --no-github` from the project directory. This
  uploads the source directly. (A starter template needs no GitHub either:
  use `create_server_from_template`.)
- **Installing the GitHub App.** `connect_github` opens the install page (or
  returns a link), but the user must finish the install on GitHub.
- **Plans and billing.** Tools report plan limits when they hit them; upgrades
  happen in the dashboard.

Docs are at https://docs.manufact.com.

## Ground rules

- **Act on explicit requests.** When the user has already asked for an action,
  such as "deploy it" or "run the evals", do it without asking again. Ask only
  when the target is ambiguous or the action goes beyond what they asked.
- **Confirm before anything destructive.** Destructive tools:
  `delete_server`, `delete_eval_suite`, `disconnect_server_repository`,
  `delete_server_custom_domain`, and `promote_deployment` (it switches production
  traffic). One confirmation per action is enough.
- **Explain usage before unrequested runs.** Deployments, checklist runs, and
  eval runs can count against the organization's plan. Before starting one the
  user did not ask for, say what it will run and ask. Don't quote plan names,
  prices, or limits from memory; a tool error reports the actual plan limit.
- **Changes need a new deployment to take effect.** This applies to environment
  variables, build settings, and subdomain changes. Say so, and offer to `deploy`.
- **Don't retry blindly.** After a write with an uncertain result, inspect with
  the matching get or list tool before retrying.
- **Present links as clickable links.** This covers GitHub connect links,
  authentication links, and dashboard URLs.
