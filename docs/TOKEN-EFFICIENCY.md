# Token-efficient MCP clients

All operations, safety annotations and handlers remain available. Compact and
legacy advertise enabled tools at startup; a capable **client** indexes them
and loads individual schemas on demand. The optional focused pilot advertises
matching tools as the session searches for them. There is no universal MCP server
`stateless` switch for this and no new generic execution gateway.

## Compact catalog profile

Set `PA_MCP_TOOL_CATALOG=compact` in the MCP server's environment, or set
`"toolCatalog": "compact"` at the top level of config.json. The environment
variable takes precedence. Restart the server and reconnect the client so its
discovery cache refreshes. The default is `compact`; explicit legacy settings remain effective. Changing the profile during
a session does not change its advertised catalog.

Compact mode replaces 49 related read tools with 22 focused tools. Each requires
one `operation` selector and the original view's arguments at the top level.
The handler, required fields, defaults, bounds, errors, pagination and result
content stay the same. All writes, common flow tools and Canvas discovery tools
retain their existing names. Feature selection applies in either profile.

The full compact catalog has 293 tools, compared with 320 in legacy mode.
Both profiles remain within the reviewed payload limits. Tool count reduction
alone does not measure tokens saved by client deferral. Native client tool
search remains the larger opportunity for reducing loaded schemas.

Use the [generated migration map](TOOL_REFERENCE.md#compact-catalog-migration)
to update saved bindings and allowlists. Old names are callable by the server
but hidden from compact discovery; some hosts refuse unadvertised calls.
Use `legacy` until those bindings migrate. Clients must retain the full input
schema. Every view's fields are visible at the root, and operation-specific
branches retain requirements and defaults. Use explicit `legacy` for saved
bindings that have not been migrated.

## Focused discovery pilot

Set `PA_MCP_TOOL_CATALOG=focused` or `"toolCatalog": "focused"` in config.json,
then restart and reconnect. This is opt-in; compact remains the default.
Use it with clients that refresh `tools/list` after
`notifications/tools/list_changed`. Clients that cache their initial catalog
should use compact or legacy. Compare with the host's native tool search before
choosing this profile.

In installed-client checks, Claude Code 2.1.229 refreshed focused inventories,
while Codex CLI/app-server 0.169.0-alpha.11.1 kept its initial inventory with
the default refresh setting. Use compact for that Codex configuration. A
process-local experimental refresh flag was tested separately; it is not
enabled by this server. Verify the actual desktop/runtime configuration before
choosing focused, since the CLI checks do not establish desktop UI or model
task behavior.

The initial catalog contains `search_operations` and `describe_operations`.
Search by task or exact tool name. Search results contain labeled description
excerpts and safety annotations. Matching operations appear in `tools/list`
with their complete original descriptions, schemas, annotations and output
schemas. Inspect the full descriptor there, or use `describe_operations` with
1 to 10 exact names. Call the original name with its original arguments.
Discovery never executes a tenant operation. There is no generic executor.
Compact read aliases are also supported with their existing `operation` field.

Search is deterministic lexical matching of enabled names and descriptions.
Use `query="*"` to browse every enabled canonical operation. Pages contain at
most ten results, default five. Continue with `nextCursor`, the same query and
the same limit. Unknown names and disabled tools are refused when describing.
An ambiguous search can include both reads and writes; inspect the complete
contract and the existing confirmation requirements before acting.

The visible catalog holds at most 32 operation descriptors by default, plus
the two discovery tools. Recently used dynamic descriptors stay visible;
older ones are evicted and can be found again. Explicit configured preloads
stay visible. Saved canonical calls still use the original dispatch, even
when their descriptors are currently hidden. Feature checks, authentication,
environment resolution, input limits, validation, confirmations and error
handling still run for every operation.

Optional configuration:

```json
{
  "toolCatalog": "focused",
  "focusedCatalog": {
    "maxTools": 32,
    "preload": ["list_flows", "get_flow", "inspect_flow_versions"]
  }
}
```

`PA_MCP_FOCUSED_TOOLS` overrides preloads with comma-separated exact names.
`PA_MCP_FOCUSED_MAX_TOOLS` overrides the operation limit. The limit must be an
integer from 10 to 297 and leave ten free slots beyond the preloads. Invalid,
duplicate, unknown or disabled preload names fail startup. Even at the maximum,
the visible list stays below 300 tools. These variables affect the server
subprocess, and setup preserves valid saved focused settings.

Each transport session owns its discovery state. Account, application,
environment or enabled-feature changes clear dynamically loaded descriptors
and invalidate cursors. Enabled preloads remain. Descriptors contain static
tool metadata, not API data or resource dropdowns. Reconnecting starts a fresh
session. This profile reduces discovery payloads; it does not defer Node module
imports or claim lower process memory.

For local development, `npm run context:focused` measures ten scripted SDK
discovery tasks without authentication or tenant requests. It includes search
results, list refreshes and complete contract descriptions. The comparison is
against an eager compact catalog, not native host tool search or actual model
prompt tokens. Model selection and task quality require native-client testing.

For installed-client inventory checks in a source checkout, use Python 3.11 or
newer with `python3 scripts/check-focused-clients.py --report /tmp/focused-clients.json`.
The check uses offline fixtures, makes no model request and does not change
global client settings. It reports cache compatibility separately from model
prompt usage. The guarded startup/RSS probe is
`scripts/measure-focused-startup.py`; it requires a built main baseline and the
existing Sandbox `live-test.mjs` helper.

## Claude Code

Current Claude Code supports deferred MCP schemas by default. For an explicit
per-launch setting, use `ENABLE_TOOL_SEARCH=true claude` in POSIX shells, or set
`$env:ENABLE_TOOL_SEARCH = "true"` before launching `claude` in PowerShell.
For a persistent setting, merge `"ENABLE_TOOL_SEARCH": "true"` into the
**Claude host's** settings.json `env` object; do not replace other settings.
Putting it only in the MCP subprocess's env does not configure Claude.

Inspect `/mcp` connection status and `/context all` loaded-tool costs in a fresh
conversation. Avoid `alwaysLoad:true` for this large catalog. Current `auto`
is threshold mode; `false` is eager. Model/provider, proxy and organization
restrictions can prevent deferral; do not bypass them or change authentication
to obtain a benchmark. See the [official MCP client guidance](https://code.claude.com/docs/en/mcp#scale-with-mcp-tool-search)
and [context-cost documentation](https://code.claude.com/docs/en/features-overview#understand-context-costs).

## Claude Desktop

Where available, select **+ → Connectors → Tool access → On demand** for the
conversation. Auto also selects connectors dynamically. Verify your actual
local extension/client; the documentation does not guarantee identical loading
granularity for every integration. [Official tool-access settings](https://support.claude.com/en/articles/13730515-manage-claude-s-tool-access)

## Codex and API clients

Use the host's native MCP discovery and tool search when exposed. Keep the
complete catalog enabled so the client can retrieve the original schemas and
safety annotations. OpenAI API `tool_search` and `defer_loading` options belong
in a supported API client request, not in this MCP server's configuration.
See the [official OpenAI documentation](https://developers.openai.com/api/docs/guides/tools-tool-search).

## Keep reference material out of startup

`CLAUDE.md` keeps workflow and safety rules, plus an ordinary link to
[the complete tool reference](TOOL_REFERENCE.md). Read only the needed section.
Do not automatically import the reference with `@path` or paste the entire
catalog into each prompt. Existing human-facing references and the full generated
reference still retain every current tool description. Generation fails if a registered tool is ungrouped or
the reference drifts.

## Ask for the smallest sufficient result

Existing options, unchanged by this release:

- Flow overview: `get_flow` with `format:"summary"`; full source is still
  available through `format:"json"` or `"both"`.
- Failure triage: `get_run_actions` with `failedOnly:true` or `actionName`,
  adding input/output payloads only when needed.
- Dataverse: select relevant fields and use `filter`/`top`.
- Component inventory: bounded `list_copilot_agent_components` without content.
- Component metadata: `get_copilot_agent_component` with `includeContent:false`.
  This still includes metadata such as name/description/schema/modified time.
- Component source: explicitly use `includeContent:true` and a suitable
  `maxContentChars`. The existing implicit default remains true/50000 for
  compatibility; no consumer behavior was silently changed.

Metadata without source has no source hash. A bounded display's hash covers the
full stored source, but a hash alone is not proof that the maker inspected that
source. Obtain complete source before a source-guarded edit. Preserve incomplete,
unavailable, permission-denied and truncated-result distinctions.

## Measure from a repository checkout

```sh
npm run context:check
npm run context:measure
npm run context:measure -- --baseline-ref <full-40-character-local-commit-sha>
npm run docs:check
npm test
```

The measurement script imports the registry offline. It does not start the
server, authenticate, invoke handlers or call tenant APIs. It reports **UTF-8
bytes, not model tokens**. Catalog and initialize-instruction budgets are separate
from automatically loaded project instructions. Optional baseline comparison
reads CLAUDE.md from an existing local commit; it does not fetch or change Git.

Regression tests pin advertised names, input schemas, safety annotations, named
dispatch, and all 128 feature combinations. This increment also makes three
discovery descriptions agent-neutral and renames one runtime diagnostic field
(see the migration note below), so the full catalog and every output are not
byte-for-byte unchanged. Future intentional changes require reviewing fingerprints
and counts; never refresh them just to hide lost capabilities.

## Review context growth before merging

`npm run context:check` reads `.github/mcp-context-budgets.json` and exits
nonzero when a reviewed byte limit is exceeded or a scenario names a missing
tool. Both startup profiles use the same byte limits and nine scenarios; compact
mode also fails at 300 tools. Scenario names map to complete grouped descriptors
without advertising the old aliases. CI runs this gate in addition to the existing contract and feature tests.
The same gate also runs in the unit suite used by local release checks.

The limits cover the full `tools/list` result, names plus initialization
instructions, each complete tool definition, each description, and nine task
selections: flow triage, Copilot source review, Canvas control discovery,
SharePoint content, Dataverse data, environment administration, Power Pages
content, Power Pages management and AI Builder prompt versions/tests. Each selection includes the original full
schemas and safety annotations, plus every tool name and the server guidance.
These selections measure payload size; they do not execute the selected tools.

The report includes current catalog, contract and safety fingerprints. Existing
contract tests independently protect all enabled tools and all 128 feature
combinations. Protocol tests verify every feature profile, a successful local
call, invalid-argument refusal and an unchanged catalog after both calls.
Authentication and network requests are forbidden in these offline tests.

All reported sizes are exact UTF-8 bytes. The names-and-instructions startup
figure and selected task payloads are client context proxies, excluding host
wrappers, search overhead, caching and conversation history. Measure actual
model tokens in the native client acceptance step below.

When adding a feature, review the context report with the feature's contracts
and task checks. Adjust a budget only when the additional size is justified.
Keep full source retrieval, continuation indicators, confirmations and every
existing capability available. Never remove schema fields or narrow the tool
catalog to make the budget pass.

## Diagnostic-output migration for the next release

Consumers parsing `get_copilot_agent_evaluation_connections` must now read
`verification.agentExecution`. The [release changelog](https://github.com/rcb0727/powerplatform-mcp-docs/blob/main/CHANGELOG.md)
documents the exact removed-key migration. The value remains `"not_verified"`:
connection metadata does not prove agent execution. The old key is removed without
an alias. Review parsers, dashboards, fixtures and assertions before upgrading;
this is an explicit output-compatibility change, not a new runtime success claim.
Tool names, arguments and permissions are unchanged.

Generic Jira evidence adapters remain available. No particular agent, tenant or
ticket is configured in the packaged runtime by this change.

## Read-only live stdio verification

Build and run only after selecting an existing non-sensitive component with
source longer than 100 characters and at most 200000 characters. Set these
environment variables in the calling shell:

- `PA_MCP_E2E_COPILOT_ENVIRONMENT`: exact environment ID.
- `PA_MCP_E2E_COPILOT_AGENT_ID`: existing parent agent GUID.
- `PA_MCP_E2E_COPILOT_COMPONENT_ID`: that agent's existing component GUID.
- Optionally `PA_MCP_E2E_CONFIG`: an existing server configuration path.
- The usual server configuration/authentication must already be available.

Then run `npm run test:e2e:context` (POSIX). In PowerShell, run
`npm run build`, set `$env:PA_MCP_E2E_CONTEXT = "1"`, then
`npx vitest run --config vitest.e2e.config.ts tests-e2e/mcp-context.e2e.test.ts`.

This explicitly opted-in suite uses the real built stdio server. It verifies
initialize guidance, the whole catalog, a session-only environment selection,
metadata/bounded/full component reads and unknown-argument refusal. It never
publishes, evaluates or changes tenant source/data. Missing targets after opt-in,
authorization errors and transport errors fail rather than becoming successful
skips. Raw server/authentication errors and tenant source are withheld from test
failure output. No package publish or release is performed.

## Native-client acceptance is a separate layer

For the same model/client and prompt, compare fresh eager and deferred
conversations. Record actual loaded schemas, search calls, successful named tool
calls, complete-loop tokens (including cache creation/read separately), latency
and discovery misses. A smaller CLAUDE.md or passing stdio test is not proof of a
specific model-token reduction. Search may add a round trip, and discovered
schemas can remain in conversation history.

Do not silently switch between the two servers when one denies access: matching
tool names do not make their authentication or effective authorization identical.
No global client setting is changed by installing this server.

The checkout-only `docs/CONTEXT-VALIDATION-20260907.md` records this increment's
measured bytes, live stdio coverage and the unresolved native-client benchmark.
It is intentionally excluded from the npm package; the complete reference and
this client guide are packaged.
