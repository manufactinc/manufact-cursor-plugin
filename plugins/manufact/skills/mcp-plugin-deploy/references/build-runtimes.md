# Build and runtime detection

Manufact honors the project's own configuration. It detects the runtime from
the repository instead of requiring a Manufact-specific config file.

## Runtime

| Runtime | Detected from | Version |
| --- | --- | --- |
| Node.js | `package.json` plus a lockfile | Node 20, 22, or 24, chosen from `engines.node`. Defaults to 22. |
| Python | `pyproject.toml` or `requirements.txt` | Python 3.11 to 3.13, chosen from `requires-python`. Defaults to 3.12. |
| Anything else | A committed `Dockerfile` (set `dockerfilePath` if it's not at the root) | Whatever the image contains. |

Deno is not detected automatically. A repository with only `deno.json` fails
with exit code 127, so add a `Dockerfile`.

## Package manager (Node)

| Lockfile | Package manager |
| --- | --- |
| `pnpm-lock.yaml` | pnpm |
| `bun.lock` or `bun.lockb` | bun |
| `yarn.lock` | yarn |
| `package-lock.json`, or no lockfile | npm |

The pnpm version comes from the `packageManager` field in `package.json` when
set. Otherwise it follows the lockfile version: lockfile 9 uses pnpm 10.

pnpm 10 and later block dependency build scripts unless the package is listed
in `onlyBuiltDependencies`, in `package.json` or `pnpm-workspace.yaml`. Running
`pnpm approve-builds` locally adds them.

- **pnpm 10:** skips unlisted build scripts. The build succeeds, but the server
  can crash at runtime with a missing native binary, typically from `esbuild`,
  `sharp`, or `protobufjs`.
- **pnpm 11:** fails the build with `ERR_PNPM_IGNORED_BUILDS`.

## Python package manager

Manufact uses uv, poetry, pipenv, or pip, whichever matches the project files.

## Framework presets and defaults

Detected presets include mcp-use, the official TypeScript and Python MCP SDKs,
FastMCP, tmcp, xmcp, and Skybridge.

| Preset family | Build | Start | Port |
| --- | --- | --- | --- |
| Node, including mcp-use | The `build` script, when present | The `start` script | 3000 |
| Python MCP SDK | Dependency install | Inferred from `pyproject.toml` | 8000 |
| FastMCP | `uv sync`, or `pip install .` | Inferred from `fastmcp.json` | 8000 |

Override any of these with `create_server` arguments, or later with
`update_server_deployment_settings`. The overridable settings are
`buildCommand`, `startCommand`, `port`, `rootDir`, `dockerfilePath`, and
`region`. Overrides are useful when the server is a sub-package, for example
`npm run mcp:build` / `npm run mcp:start`.

## Injected environment

- **`PORT` and `HOST`:** the server must listen on these. `HOST` is `0.0.0.0`.
- **`MCP_URL`:** the deployment's public MCP URL. It's useful for absolute links
  and widget asset URLs.
- **User-defined variables:** these come from the server's environment
  variables, scoped to production or to one preview branch. Changes apply on the
  next deployment.

## Health check

After starting the container, Manufact sends a JSON-RPC `initialize` request to
`<base>/mcp` via POST. It waits for a short stability window, then routes
traffic. Deployments are blue-green: if the check fails, the previous
production deployment keeps serving.
