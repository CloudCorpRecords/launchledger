# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted Launchelion MCP server, a catalogue of live, launched AI tools. The four captured tools of the labelled subset are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `search_tools`, `get_tool`, `list_alternatives`, `list_offers`. Two captured tools are deliberately omitted: `submit_tool`, which submits a new listing on behalf of its maker carrying the maker's email and returns a claim/sign-up URL, and `create_checkout`, which creates a Stripe checkout URL for a paid listing add-on and so originates a purchase flow. `list_offers` is kept: it only reads the published add-on price list and buys nothing itself.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `launchledger error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `LAUNCHELION_MCP_URL`. The registry record declares no headers on the remote, so no auth header is ever sent.
