# Publishing checklist checks

Each check has a `checkId`, a `status` (`pass`, `fail`, `skip`, or `pending`),
a `severity` (`error`, `warning`, or `info`), a `category`, and the `platforms`
it applies to. An empty platform list means the check applies to every client.

A platform is **ready** when no check that applies to it has `status: fail` with
`severity: error`.

For mcp-use servers, most fixes go in the `server.tool({ ... })` definition:

- `title`
- `description`
- `annotations: { readOnlyHint, destructiveHint, openWorldHint }`

Widget settings go in `widgetMetadata`. Other frameworks have equivalent fields
on the tool and resource definitions.

## Protocol and authentication

| Check | Severity | Platforms | What it needs |
| --- | --- | --- | --- |
| `server-responds` | error | all | MCP `initialize` succeeds at the target URL |
| `auth-required` | error | all | The checklist can connect with valid, refreshable OAuth. Fix with `authenticate_deployment`. |
| `server-tools-capability-declared` | error | all | The initialize response declares the `tools` capability |
| `tools-list-responds` | warning | all | `tools/list` returns results |
| `resources-list-responds` | info | all | `resources/list` responds |

## Tool metadata

| Check | Severity | Platforms | What it needs |
| --- | --- | --- | --- |
| `tool-hints-present` | error | chatgpt | Every tool sets boolean `readOnlyHint`, `destructiveHint`, and `openWorldHint` |
| `claude-tool-hints-present` | error | claudeai | Every tool sets boolean `readOnlyHint` or `destructiveHint` |
| `tool-title-present` | error | claudeai | Every tool has a top-level `title`. `annotations.title` is only a legacy fallback. |
| `tool-name-length` | error | claudeai | Tool names are at most 64 characters |
| `tool-names-unique` | error | chatgpt | No duplicate tool names |
| `tool-input-schema-type` | error | all | `inputSchema` is an object with `type: "object"` |
| `tool-output-schema-valid` | error | all | A declared `outputSchema` is structurally valid, for example every name in `required` exists in `properties` |
| `tool-input-header-annotations-valid` | error | all | `x-mcp-header` annotations on inputs are valid |
| `openai-file-params-schema-valid` | error | chatgpt | Each `openai/fileParams` field is a file object with `download_url` and `file_id` |
| `tool-description-present` | warning | all | Every tool has a description |
| `tool-name-format` | warning | all | Names use only letters, digits, `_`, `-`, and `.` |
| `tool-name-protocol-length` | warning | all | Names are 1 to 128 characters |
| `tool-text-hidden-characters` | warning | all | No hidden control characters in names or descriptions |
| `claude-code-metadata-length` | warning | claude-code | Each tool description and the server instructions are at most 2 KB, with critical guidance first |
| `cursor-tool-name-budget` | warning | cursor | Server alias plus tool name fit about 60 characters |

## MCP apps and widgets

These checks are skipped when the server has no widget (view) resources.

| Check | Severity | Platforms | What it needs |
| --- | --- | --- | --- |
| `resource-content-valid` | error | chatgpt, claudeai | The widget resource envelope is valid |
| `tool-resource-uri-readable` | error | chatgpt, claudeai | The resource URI a tool links to can be read |
| `tool-visibility-valid` | error | chatgpt, claudeai | Tool visibility metadata is valid |
| `view-domain-present` | error | chatgpt | Widgets declare one exact HTTPS origin (`_meta.ui.domain`), with no path. All widgets share it. |
| `tool-resource-metadata-complete` | warning | chatgpt, claudeai | A `ui://` URI and the `text/html;profile=mcp-app` MIME type. In mcp-use, set the widget description in `widgetMetadata`. |
| `view-csp-origin-coverage` | warning | chatgpt | Every external origin appears in the matching `_meta.ui.csp` list: `connectDomains`, `resourceDomains`, `frameDomains`, `baseUriDomains`, or `redirectDomains` |
| `view-domain-responds` | warning | chatgpt, claudeai | The declared domain responds |
| `view-resource-coverage` | warning | chatgpt, claudeai | Every discovered widget resource was inspected |
| `tool-invocation-metadata-present` | warning | chatgpt | `openai/toolInvocation` has `invoking` and `invoked` status strings |

## Skills over MCP

| Check | Severity | Platforms | What it needs |
| --- | --- | --- | --- |
| `skills-over-mcp-readable` | error | chatgpt | Skills are readable and within ChatGPT's limits: at most 5 skills, each `SKILL.md` at most 256 KB, at most 100 resources per skill, 5 MB total |

## Policy review

An automated reviewer compares the server's listing and behavior with current
ChatGPT and Claude directory policies. A clear violation is an `error`; a
likely problem is a `warning`.

Common findings:

- Descriptions that overclaim or mislead.
- Collecting personal or sensitive data without a clear need.
- Destructive actions with no confirmation.
- Oversized initial results from Claude MCP apps: around 150,000 serialized
  characters or more triggers a warning.

## End-to-end checks

These run the server live in real ChatGPT and Claude.ai sessions.

| Check | Severity | What it needs |
| --- | --- | --- |
| `e2e-chatgpt-tool-call` | error | ChatGPT can call one of the server's tools end to end |
| `e2e-claudeai-tool-call` | error | Claude.ai can call one of the server's tools end to end |
| `widget-renders-on-chatgpt` | warning | Widgets render in ChatGPT |
| `widget-renders-on-claudeai` | warning | Widgets render in Claude.ai |

End-to-end failures don't count toward `score`, but a failed tool call still
makes that platform not ready. Protected servers must pass `auth-required`
first.
