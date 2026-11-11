# Analytics query patterns

Start with the live catalog and keep the returned contract authoritative.
The examples below show how to choose a query; use the declared tool schemas
for exact arguments and the catalog for supported metric and column names.

## Scope and comparison

Resolve relative requests such as "yesterday" in the user's timezone, then
send explicit UTC timestamps when using a custom window. Label the timezone
in the answer. For a period comparison, use equal-duration, non-overlapping
windows and identical filters. Compare complete periods when the user asks
about full days; flag a current day that is still in progress.

For focused summary reads, timeRange supports 24h, 7d, 30d, or 90d and defaults
to 30d. Typed metrics require window, either a preset (1h, 24h, 7d, 30d, 90d)
or an object with from and to timestamps. Windows are half-open UTC [from, to),
to cannot be in the future, and retention can clamp the start. SQL timeRange
uses the same window objects and defaults to the last 7d when omitted.

Omitting branch includes all environments for analytics; it does not select
production. Explicit branch selects that preview branch. The runtime log
lookup instead selects production when branch is omitted. The catalog lists
branches and their gateway subdomains; use its mapping when grouping by
subdomain rather than inventing a branch URL.

Legacy clients and tools selections use exact names. Omitted or empty clients
use the default non-internal traffic selection; explicit client names can
include internal traffic. A tools filter selects interactions that called
any selected tool, including other activity in those interactions. It is not
equivalent to filtering events by resource name. Check the catalog and each
focused tool's schema for their supported filter semantics.

Typed filters use a catalog dimension, an operator (eq, neq, in, not_in, like),
and values. The default segment is mcp_traffic; segments are ANDed. Selecting
all permits non-MCP HTTP in the population. An eq/in client_name filter forces
internal traffic to be included so an explicit internal client can be selected.
sessionTools selects sessions that used the named tools; filtering the tool
dimension selects matching tool events. Preserve that distinction and inspect
meta.semantics for any tool_events fallback.

Issue lists cover all environments and accept no branch filter. Issue detail
counts and occurrences cover the issue's lifetime rather than the list's
analytics window. Do not label issue detail as evidence for a branch-specific
or period-specific total.

Session lists also cover all environments and have no branch argument.
Their client, tool, country, and actor filters narrow the list. Session detail
covers the interaction's lifetime across environments; it is not limited to
the list's analytics window. Captured request/result summaries can be absent
or truncated. Do not infer uncaptured payload contents.

## Question examples

| User question | Useful read and interpretation |
| --- | --- |
| "How much was this server used this month?" | Read the summary for an overview. If a calendar month is intended, use a custom typed metrics window from the catalog, rather than substituting a rolling 30d period. State whether counting requests, tool calls, interactions, or actors. |
| "Which tools fail most often for Claude?" | Discover the relevant error and call metrics, client filter, and resource grouping. Query failures and denominator together; show failure count and rate, with the same metric population and time window. Fetch issue or event evidence for the largest relevant failure groups. |
| "Has latency improved since our release?" | Query the same catalog-supported latency metric in windows before and after the release with matching client, environment, and resource filters. Show request counts and approximation metadata alongside percentiles. A deployment timestamp establishes timing, not causation. |
| "Why did this session fail?" | Find the relevant session or interaction, then read its events and any associated issue detail. Use `get_server_analytics_error_runtime_logs` with the observed event timestamp if runtime context is needed. Its window is two minutes either side of the event and its log-entry cap is 5000. |
| "Which users came back each week?" | Discover the actor and interaction datasets and definitions. Prefer typed distinct metrics when supported. For a cohort or join that requires SQL, use only permitted catalog fields; preserve verified versus approximate identity and retention limits. |

### Errors rose on a preview branch

Keep branch in the metrics and SQL request scope. For a last-7d increase,
query errors, events, and error_rate with compare: previous_period first;
then group by tool and filter mcp_method = tools/call to isolate tool failures.
Compare both counts and rates: more calls can raise errors without worsening
the failure ratio. Add daily buckets to locate when the increase began.

The session and issue lists cannot scope to a branch. To identify affected
interactions on that branch, use catalog-supported raw metrics grouped by
interaction_id, or a small SQL events read with the same branch and time
scope. Use the error spike's interval and rawWindowHintDays to keep the read
within budget. Read only the chosen interaction IDs with the session detail
tool; its events are lifetime evidence across environments. Preserve the
branch-scoped event timestamps as evidence and do not attribute the detail's
entire lifetime error count to that branch. If SQL is unavailable and a typed
raw query exceeds budget, report the unverified session attribution.

### Typed metrics calls

Replace the serverId marker with a real returned server ID. These examples
use the current catalog vocabulary; re-check it before sending the call.

Tool calls and interactions by day with the preceding-period comparison,
using `query_server_analytics_metrics`:

```json
{
  "serverId": "<returned server ID>",
  "window": { "preset": "7d" },
  "measures": ["tool_calls", "sessions"],
  "granularity": "day",
  "compare": "previous_period"
}
```

Top ten tools by errors, with the tool-call denominator and error ratio:

```json
{
  "serverId": "<returned server ID>",
  "window": { "preset": "7d" },
  "measures": ["errors", "tool_calls", "error_rate"],
  "dimensions": ["tool"],
  "filters": [{ "dimension": "mcp_method", "op": "eq", "values": ["tools/call"] }],
  "orderBy": [{ "field": "errors", "dir": "desc" }],
  "limit": 10
}
```

Typed error_rate is errors/events on a 0–1 scale. The method filter makes
events and tool_calls share a denominator in this example. Tool-result
failure rate uses tool_result_errors/tool_outcome_events and includes only
events with a known parsed result; choose it for a question about isError.

Verified versus IP-derived actor estimates:

```json
{
  "serverId": "<returned server ID>",
  "window": { "preset": "30d" },
  "measures": ["verified_actors", "approximate_actors"]
}
```

### Read-only SQL call

For SQL, prefer the typed result's meta.sql as an already compatible starting
point. The dialect is ClickHouse. One SELECT or UNION is allowed; SETTINGS,
FORMAT, INTO, table functions, and non-catalog tables are rejected. The
request scope enforces server, branch, internal-traffic exclusion, and
retention. Time predicates still belong in the query: bind its window with
{from:DateTime64(3)} and {to:DateTime64(3)}, filtering raw events on time and
rollups on bucket.

Read the narrowest relation with the required columns. Prefer clients_daily
for client-level headline metrics, sessions_daily when country is needed,
tool_health_daily for tool health, and metrics_daily for wider dimensions.
Use sessions_hourly or metrics_hourly for short windows or hourly buckets.
Use tool_interactions for sessions that used a tool; events is the bounded
raw escape hatch. Consult the catalog's sources, row budgets, and examples.

Rollup rows are pre-aggregated cells: count with sum(events), never count().
Compute average duration from sum(duration_sum)/sum(duration_n), and merge
sketches rather than counting rows or summing distinct estimates.

This `query_server_analytics_sql` call reads total requests and approximate
interactions by client for a week:

```json
{
  "serverId": "<returned server ID>",
  "timeRange": { "preset": "7d" },
  "maxRows": 50,
  "sql": "SELECT client_name, sum(events) AS requests, uniqCombined64Merge(14)(interactions) AS sessions FROM clients_daily WHERE bucket >= {from:DateTime64(3)} AND bucket < {to:DateTime64(3)} AND traffic_class != 'non_mcp' GROUP BY client_name ORDER BY requests DESC LIMIT 50"
}
```

The stored sketch precision is 14 for sessions and actors. Percentiles use
quantileTDigestMerge(level)(duration_q), or quantilesTDigestMerge for several
levels. Use the current catalog expressions rather than guessing a merge
function. Raw lists can contain ingest retry copies; when exact event rows
matter, use LIMIT 1 BY id. Do not add FINAL to aggregates. For latest raw
events, filter server_id by equality, ORDER BY time DESC, and use a small
LIMIT so the engine can stop early.

For approximate actors grouped by the catalog's client_name dimension, typed
measures: [approximate_actors] and dimensions: [client_name] already suffice.
Use SQL when the user needs a custom expression, such as merging differently
cased client names into one group. On clients_daily, that expression is:

```sql
SELECT lowerUTF8(client_name) AS normalized_client,
       uniqCombined64MergeIf(14)(actors, actor_trust = 'approximate') AS approximate_actors
FROM clients_daily
WHERE bucket >= {from:DateTime64(3)} AND bucket < {to:DateTime64(3)}
  AND traffic_class != 'non_mcp'
GROUP BY normalized_client
ORDER BY approximate_actors DESC
LIMIT 50
```

Send that SQL with a custom timeRange: {from, to} and the chosen branch,
includeInternal, and maxRows scope. Merge the sketch across cells so the same
actor in different versions or case variants is counted once within the
group. This remains a sketch estimate of IP-derived actors; SQL does not
convert it into verified identities or exact people. SQL response columns do
not carry the typed accuracy flags, so explain approximation from the catalog
measure and merge function. Do not sum these group counts as a global total.

## Pagination and completeness

List tools return summaries. Follow the returned next page or cursor using
the same server and filters until the relevant records are found or the
requested population is covered. Preserve returned identifiers for detail
calls. A first page is not an exhaustive list; do not derive total usage from
its row count. Cursor tokens are opaque and tied to the selected query scope.

Typed metrics and SQL return bounded result sets. Respect catalog limits for
window size, groupings, filters, result rows, query length, and execution.
Use aggregation, a smaller time window, or fewer dimensions when needed.
Check truncation and partial-result metadata before describing a result as
complete. Dashboard sections can fail independently; an unavailable section
does not make another section's successful data disappear or become zero.
Drill meta.partial.reason can be row_cap (the newest rows within the cap) or
raw_budget (a narrowed window); use meta.effectiveWindow when provided.

Session lists use an opaque nextCursor with a default limit of 25 and maximum
100. Session event detail uses local limit/skip slicing over the backend's
returned events, defaults to 25, and returns at most 50 in one tool response.
Each payload preview is capped at 1024 encoded UTF-8 bytes. Use nextSkip to
page within availableEvents; an upstream nextCursor is informative and cannot
be passed back to the detail endpoint. This does not guarantee that the
backend returned the interaction's entire history. Issue lists use
limit/skip pagination. Check their returned pagination and truncation metadata.

The current catalog limits typed queries to 12 measures, 4 dimensions, 10
filters, 100 values per filter, 50 sessionTools, and topN at most 50. Result
limit defaults to 200 and cannot exceed 1000. Omit granularity for totals;
auto chooses at most 240 buckets. Requested minute buckets over 6h or more
than 1500 buckets are coarsened; read meta.granularity.effective. Weeks start
on Monday. Limits may evolve, so use the live catalog when it differs.

SQL text is limited to 16384 bytes; maxRows defaults to 100 and cannot exceed
500. SQL admission allows two concurrent requests per organization. Dashboard
batch accepts at most 12 named queries and cards combined. Raw event reads
have a 20-million-row / 2-GiB read budget; the catalog's rawWindowHintDays
helps choose a window that fits. Read freshness metadata when assessing how
current the telemetry is.

## Recoverable errors

- Validation or unsupported metric/filter: re-read the catalog, correct the
  query, and preserve the user's intended question.
- Retention/window limit: report the available window and use it when it still
  answers the question. Do not silently substitute a different period.
- SQL rejection: correct the read-only syntax or disallowed dataset, column,
  or expression from the catalog guidance. Do not retry with an unrestricted
  backend or alternate credentials.
- SQL unavailable or disabled: continue with supported typed/focused reads
  when possible and state the missing capability. Do not assume every hosted
  Cloud environment has enabled the SQL reader.
- Timeout, excessive groups, rows, or query cost: narrow the window, aggregate
  more, or split into meaningful windows within the reported limits. Distinct
  counts and percentiles cannot generally be summed or averaged across splits.
- Missing server or access denied: verify the server ID and chosen organization.
  The connected OAuth identity is authoritative; do not request credentials.
- Transient backend error: retry a bounded read once if appropriate. If it
  still fails, report which evidence is unavailable and stop that query.
- Rate limit or admission rejection (429): use supplied retry guidance,
  reduce concurrency, and retry a read once after the indicated delay. Do not
  launch a burst of duplicate SQL queries.

If the response reports a partial or truncated result, distinguish what it
supports from what remains unknown. A failure or warning is not an analytical
result, and no-data responses do not establish zero activity outside the
effective retained scope.
