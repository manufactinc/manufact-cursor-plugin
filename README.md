# Manufact Cursor plugin

Cursor Marketplace plugin for building, testing, and deploying MCP servers and apps with [mcp-use](https://github.com/mcp-use/mcp-use) and [Manufact](https://manufact.com).

## What's included

The [`manufact`](plugins/manufact) plugin bundles:

- **Manufact MCP server** (`https://mcp.manufact.com/mcp`) — deploy and manage MCP servers on Manufact (OAuth)
- **mcp-use docs MCP server** (`https://docs.mcp-use.com/mcp`) — search the mcp-use SDK documentation
- **`mcp-apps-builder` skill** — mcp-use best practices for building MCP servers and apps

See [plugins/manufact/README.md](plugins/manufact/README.md) for details.

## Local development

```bash
ln -s "$(pwd)/plugins/manufact" ~/.cursor/plugins/local/manufact
```

Then restart Cursor (or run **Developer: Reload Window**).

## Validation

```bash
node scripts/validate-template.mjs
```

## Updating the bundled skill

The `mcp-apps-builder` skill is vendored from [mcp-use/mcp-use](https://github.com/mcp-use/mcp-use). To refresh it, copy `skills/mcp-apps-builder` from that repo into `plugins/manufact/skills/`.
