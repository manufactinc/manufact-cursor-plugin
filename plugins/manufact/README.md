# Manufact plugin for Cursor

![Manufact](assets/icon.svg)

**Build, test & ship MCP servers**

Manufact helps you build, deploy, test, and monitor MCP servers and plugins for the most popular AI Agents. Deploy from GitHub or a starter template, inspect deployment status and logs, configure servers, test behavior with evals in supported clients, analyze usage and errors, check publishing readiness for supported directories, and package plugins with guided listing and review preparation. Connect your Manufact account to use these workflows.

## Plugin details

| Field | Value |
| --- | --- |
| Publisher | Manufact |
| Category | Developer Tools |
| Version | 2.2.0 |
| Capabilities | Read, Write |
| Website | [manufact.com](https://manufact.com) |
| Support | [Manufact support](https://manufact.com/support) |
| Privacy policy | [Manufact privacy policy](https://manufact.com/privacy) |
| Terms of service | [Manufact terms](https://manufact.com/terms) |
| Plugin source | [manufactinc/manufact-cursor-plugin](https://github.com/manufactinc/manufact-cursor-plugin) |
| MCP server source | [mcp-use/manufact-mcp-server](https://github.com/mcp-use/manufact-mcp-server) |

The skills, references, shared listing information, and PNG/SVG icons are synchronized with the Manufact OpenAI plugin 2.2.0. Cursor uses `assets/icon.svg` as its logo; `assets/icon.png` is the matching raster icon. Cursor discovers skills from `skills/` and MCP connections from `mcp.json`.

## MCP servers

| Server | URL | What it does |
| --- | --- | --- |
| `manufact` | `https://mcp.manufact.com/mcp` | Deploy, configure, test, and monitor MCP servers and plugins on [Manufact](https://manufact.com). Connect your account through Cursor's OAuth flow on first use. |
| `mcp-use-docs` | `https://docs.mcp-use.com/mcp` | Search and read the [mcp-use](https://docs.mcp-use.com) SDK documentation — API references, guides, and code examples. |

Installing the plugin does not deploy a server or start evals. Ask the agent for the workflow you need. Available eval clients and publishing targets come from the live Manufact tools.

## Skills

| Skill | When to use it |
| --- | --- |
| [manufact-getting-started](skills/manufact-getting-started/SKILL.md) | Learn the account model, inspect existing servers, and choose a starter deployment or the next workflow. |
| [mcp-plugin-deploy](skills/mcp-plugin-deploy/SKILL.md) | Deploy from GitHub or a starter template, configure builds, environments and domains, or investigate failed deployments. |
| [mcp-plugin-testing](skills/mcp-plugin-testing/SKILL.md) | Discover supported clients and models, create and run eval suites, and investigate failures. |
| [mcp-plugin-analytics](skills/mcp-plugin-analytics/SKILL.md) | Analyze usage, errors, latency, clients, actors, and sessions using scoped evidence and catalog-driven metrics. |
| [mcp-plugin-publishing](skills/mcp-plugin-publishing/SKILL.md) | Check publishing readiness for supported directories and prepare Agent Plugin packages, listing metadata, review cases, and demos. |
| [mcp-apps-builder](skills/mcp-apps-builder/SKILL.md) | Build MCP servers and apps with mcp-use: tools, resources, prompts, widgets, security patterns, and deployment. |

The builder skill is retained from [mcp-use/mcp-use](https://github.com/mcp-use/mcp-use) and is also installable standalone:

```bash
npx skills add https://github.com/mcp-use/mcp-use --skill mcp-apps-builder
```

## Try it

- "Help me get started with Manufact."
- "Show the status of my MCP servers."
- "Test my MCP server in supported AI agents."
- "Check whether my MCP server is ready to publish."
- "Create an MCP server that exposes my REST API."
- "Add a widget to my MCP server."
- "Deploy this MCP server to Manufact."
- "How do I use sampling in mcp-use?"

## Documentation

- [Manufact docs](https://docs.manufact.com)
- [mcp-use docs](https://docs.mcp-use.com)
- [mcp-use on GitHub](https://github.com/mcp-use/mcp-use)
