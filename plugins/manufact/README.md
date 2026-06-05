# Manufact plugin for Cursor

Everything you need to build, test, and deploy MCP servers and apps from Cursor.

## What's included

### MCP servers (`mcp.json`)

| Server | URL | What it does |
| --- | --- | --- |
| `manufact` | `https://mcp.manufact.com/mcp` | Manage your [Manufact](https://manufact.com) projects: deploy MCP servers, inspect logs, and manage your cloud workspace. Authenticates via OAuth on first use. |
| `mcp-use-docs` | `https://docs.mcp-use.com/mcp` | Search and read the [mcp-use](https://docs.mcp-use.com) SDK documentation — API references, guides, and code examples. |

### Skills

- **`mcp-apps-builder`** — mcp-use framework best practices for building MCP servers and apps: tools, resources, prompts, widgets, security patterns, and deployment. The agent consults this skill whenever you work on an MCP server. Sourced from [mcp-use/mcp-use](https://github.com/mcp-use/mcp-use); also installable standalone with:

  ```bash
  npx skills add https://github.com/mcp-use/mcp-use --skill mcp-apps-builder
  ```

## Usage

After installing the plugin, try:

- "Create an MCP server that exposes my REST API"
- "Add a widget to my MCP server"
- "Deploy this MCP server to Manufact"
- "How do I use sampling in mcp-use?"

## Links

- [Manufact](https://manufact.com)
- [mcp-use docs](https://docs.mcp-use.com)
- [mcp-use on GitHub](https://github.com/mcp-use/mcp-use)
