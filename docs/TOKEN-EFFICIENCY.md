# Token-efficient MCP clients

All tool names, input schemas, safety annotations and handlers remain available. These
servers advertise their full feature-enabled catalog; a capable **client** indexes
it and loads individual schemas on demand. There is no universal MCP server
`stateless` switch for this and no new generic execution gateway.

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
