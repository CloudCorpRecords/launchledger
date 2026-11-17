# BUILD REPORT — launchledger

Clean-room rebuild of `com.launchelion/launchelion` v1.0.0
(remote-only MCP server at `https://launchelion.com/api/mcp/mcp`),
built 2026-10-28 as a forwarding client. Package: `@reneturcios/launchledger`
0.1.0, registry name `io.github.CloudCorpRecords/launchledger`.

## Surface built

Labelled subset: **4 of 6 captured tools built**, 2 omitted (see below).

| Tool | Required fields as served |
|------|---------------------------|
| `search_tools` | `query` |
| `get_tool` | `slug` |
| `list_alternatives` | `target` |
| `list_offers` | *(none — no `required` key in the capture, served verbatim)* |

Each served tool carries exactly the keys `name`, `title`,
`description`, `inputSchema`, with the `inputSchema.$schema` marker
(`http://json-schema.org/draft-07/schema#`) exactly as captured.
`search_tools` / `get_tool` / `list_alternatives` each carry one
string property (`query` / `slug` / `target`, `minLength: 1`, with
the captured descriptions) and their captured `required` arrays;
`list_offers` carries empty `properties` and no `required` key. No
tool carries `additionalProperties` — none was captured, none was
invented. The tool literals were generated programmatically from
`harness/fixtures/launchelion-tools-original.json` by the script at
`/tmp/laneb5-launchledger-scratch/gen.py` (parse + re-serialise into
`/tmp/laneb5-launchledger-scratch/tool_defs.txt`, spliced into
`src/forward.ts` between the TOOL_DEFS_GENERATED markers) — never
retyped — and the generator's own round-trip check, the
registration-integrity test suite, and the parity harness each
confirmed the served literals deep-equal the fixture's 4 tool
objects, in order. Tools are registered via the MCP SDK low-level
Server API (`ListToolsRequestSchema` / `CallToolRequestSchema`
handlers), not McpServer+zod.

## Omissions — `submit_tool` and `create_checkout`

- OMITTED: `submit_tool` — submits a new AI tool to Launchelion on
  behalf of its maker, requiring the maker's `email`, creating a
  draft listing plus a claim/sign-up URL: a submission carrying a
  person's email (lead-routing/submission family).
- OMITTED: `create_checkout` — creates a Stripe checkout URL for
  one paid add-on on one listing: it originates a purchase/checkout
  flow, per the standing never-rebuild-checkout rule.
- Absence is asserted in tests at three levels for BOTH tools:
  registry level (`TOOL_DEFS` / `TOOL_NAMES` / `isKnownTool`), MCP
  layer (an in-memory-transport `tools/list` response carries
  exactly the 4 built tools and deep-equals the fixture;
  `handleCallTool` and an MCP `tools/call` for each omitted tool
  are rejected before any network hop), and CLI level
  (`launchledger submit_tool '{}'` and
  `launchledger create_checkout '{}'` each exit 2 with usage and
  are not listed among the commands).
- KEPT: `list_offers` — a published price read (add-on name,
  one-sentence effect, price in USD). Reading a published price
  list is keepable even though the prices belong to the omitted
  checkout's products; the kept surface's core (catalogue search /
  fetch / alternatives) is a faithful standalone catalogue. Its
  captured description mentions `create_checkout` in prose; that
  text is served verbatim as captured, and no checkout tool is
  registered.

## Fixture conflicts

**None.** The spec and the fixture agree on the tools' names,
titles, descriptions, the `required` arrays (`query` / `slug` /
`target`, and none for `list_offers`), and the property shapes.
The fixture already excludes both omitted tools, exactly as the
spec's Omissions section describes.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://launchelion.com/api/mcp/mcp`.
- Env override `LAUNCHELION_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent, under any environment. Tests assert that no
  `Authorization` header (and no `X-API-Key` header) is sent with an
  empty environment, with a `LAUNCHELION_API_KEY`-style variable
  present, and with `AUTHORIZATION` / `API_KEY` /
  `LAUNCHELION_TOKEN` variables present.
- Error rendering `launchledger error:` text for upstream failures
  — labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP JSON-RPC:
  initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring `Mcp-Session-Id`
  when issued; plain-JSON and SSE replies are both parsed. The
  forwarding core takes an injectable fetch; tests stub the
  upstream — no live network calls in tests, and no live call is
  claimed beyond the coordinator's tools/list capture.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3), then populated with `npm ci --ignore-scripts --offline`.
- Parses as JSON; `name` == `@reneturcios/launchledger`.
- Package entries: **128**.
- sha256: `aad6401120f2775be925b71a1d6011f10c023c3b724a6d2d866a7aa385498119`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "Search a live catalogue of launched AI tools, fetch details, alternatives and paid add-ons." — **91 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/launchledger:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio. Dockerfile
  (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/launchledger"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/launchledger`
  and its smoke test asserts the server name `launchledger`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **70 tests, 70 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 4
   tools; the omission suite asserts `submit_tool` and
   `create_checkout` absent at registry, MCP-layer — tools/list and
   tools/call — and CLI level; plus session handling/reuse, env
   override, the no-Authorization-header assertions under every
   environment, verbatim argument forwarding and result pass-through
   including upstream isError, SSE parsing, error rendering,
   MCP-layer validation, and CLI behaviour).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/launchelion-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/launchledger/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/launchledger/parity-out` → **verdict PASS, parity 100.0**
   (4/4 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
