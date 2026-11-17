# Changelog

## 0.1.0 (2026-10-28)

- First release: a forwarding client for the hosted research-intel MCP server. All 5 captured tools are answered locally for tools/list, and every tools/call is forwarded upstream over a streamable-HTTP JSON-RPC session (initialize, notifications/initialized, tools/call, honouring the Mcp-Session-Id header when the upstream issues one).
- Tools: `new_funding`, `whats_changed_since`, `delta_digest`, `entity_watch`, `search_records`. The capture carries no `title` keys, so none are served; `new_funding`, `whats_changed_since`, `delta_digest` and `search_records` carry no `required` key in the capture and none is served; `entity_watch` requires `type` and `value`.
- Upstream results pass through verbatim, including the isError flag; upstream failures surface as `grantpulse error:` text results.
- One bin serves the CLI (one subcommand per tool, plus `--version`) and the stdio MCP server (`mcp` subcommand). The upstream endpoint can be overridden via `RESEARCHINTEL_MCP_URL`. The upstream record declares no headers, so no auth header is ever sent.
