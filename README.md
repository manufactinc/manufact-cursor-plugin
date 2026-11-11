# Manufact Cursor plugin

Build, deploy, test, and monitor MCP servers and plugins for the most popular AI Agents.

The [`manufact`](plugins/manufact) plugin includes the skills and branding from the Manufact OpenAI plugin **2.2.0**, alongside the existing mcp-use builder skill and documentation connection.

## What's included

- **Manufact MCP server** (`https://mcp.manufact.com/mcp`) — deploy, configure, test, monitor, and prepare MCP servers and plugins for publishing. Connect your Manufact account through Cursor's OAuth flow on first use.
- **mcp-use docs MCP server** (`https://docs.mcp-use.com/mcp`) — search and read the mcp-use SDK documentation.
- **Five Manufact skills** — getting started, deployment, testing, analytics, and publishing, with their supporting references.
- **`mcp-apps-builder` skill** — mcp-use best practices for building MCP servers and apps.

See [plugins/manufact/README.md](plugins/manufact/README.md) for the skill catalog, example prompts, and support links.

## Local development

```bash
ln -s "$(pwd)/plugins/manufact" ~/.cursor/plugins/local/manufact
```

Then restart Cursor (or run **Developer: Reload Window**).

## Validation

```bash
node scripts/validate-template.mjs
```

## Updating the bundled skills and branding

The five Manufact skills, their references, and `assets/icon.png` / `assets/icon.svg` are sourced from the Manufact OpenAI plugin 2.2.0. Keep them synchronized with future OpenAI plugin releases. The getting-started skill adapts the authentication introduction for Cursor, where skills can be discovered before the MCP connection has authenticated.

Map shared listing fields to `.cursor-plugin/plugin.json`, and keep the marketplace description and version in sync. Cursor-specific identity, repository, and connection configuration stay in this repository. Listing details without a Cursor manifest field (the full description, example prompts, and support/policy links) live in the plugin README.

The retained `mcp-apps-builder` skill is vendored from [mcp-use/mcp-use](https://github.com/mcp-use/mcp-use). To refresh it, copy `skills/mcp-apps-builder` from that repo into `plugins/manufact/skills/`.
