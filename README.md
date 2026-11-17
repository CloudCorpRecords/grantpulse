# grantpulse

What funding just landed, and what changed since you last looked? grantpulse is a local forwarding client for the hosted research-intel MCP service: it answers `tools/list` locally from the captured tool definitions and forwards every call to the service, returning the answers exactly as the service sends them back.

All five tools read two published registers — NIH grant awards and ClinicalTrials.gov registrations — and return the register records themselves, filtered by institution, principal investigator, sponsor, topic, state, NIH institute/center, or trial phase. There is no contact data, scoring, signup, or send tool in this surface.

## The tools

- **`new_funding`** — The newest NIH grants and newly-registered clinical trials, filterable by `institution`, `pi`, `sponsor`, `sponsorClass`, `topic`, `state`, `ic`, `phase`, `kind`, `keyword`, and amount range; `limit` caps rows per kind (default 50). No required parameters.
- **`whats_changed_since`** — Grants and trials that are new or modified since a `cursor` token from a prior call, with the same filters as `new_funding`. No required parameters.
- **`delta_digest`** — New grants plus new trials in a filter over the last N `days` (default 7). No required parameters.
- **`entity_watch`** — Recent funding and trial events for one entity. Both `type` (`institution`, `pi`, `sponsor`, or `topic`) and `value` are required.
- **`search_records`** — Ad-hoc search over snapshotted grants and trials by any filter, including `keyword`, `sponsorClass`, `topic`, and amount range; `limit` defaults to 50. No required parameters.

## Install and run

As an MCP server via npm:

```bash
npx -y @reneturcios/grantpulse mcp
```

MCP client configuration:

```json
{
  "mcpServers": {
    "grantpulse": {
      "command": "npx",
      "args": ["-y", "@reneturcios/grantpulse", "mcp"]
    }
  }
}
```

As a container:

```bash
docker run -i ghcr.io/cloudcorprecords/grantpulse
docker run --rm ghcr.io/cloudcorprecords/grantpulse search_records '{}'
```

## CLI

One binary serves both the command line and the MCP server. Each tool is a subcommand taking a single JSON string with that tool's arguments:

```bash
grantpulse new_funding '{"institution":"Stanford","topic":"oncology"}'
grantpulse whats_changed_since '{"cursor":"abc123"}'
grantpulse delta_digest '{"days":14}'
grantpulse entity_watch '{"type":"institution","value":"Stanford"}'
grantpulse search_records '{"keyword":"cancer","limit":10}'
grantpulse --version
```

Exit codes: `0` when the forwarded call succeeds, `1` when the upstream result is an error, and `2` for usage problems such as an unknown subcommand or a malformed JSON argument.

## Configuration

- `RESEARCHINTEL_MCP_URL` — Overrides the upstream endpoint. The default is the pinned hosted endpoint, `https://research.dropwatchhq.com/mcp`. (The override variable is a reconstruction added by this rebuild.)

The upstream registry record declares no headers, so no authentication header is ever sent; the tool surface was captured keyless.

Upstream failures — a JSON-RPC error from the service, a non-2xx response, an unreadable body, or a request that fails outright — are reported to the caller as an error result whose text starts with `grantpulse error:`.

## How forwarding works

The first forwarded call opens a session: an `initialize` request (protocol version `2025-03-26`), followed by a `notifications/initialized` notification, then the `tools/call` itself. If the service answers the initialize request with an `Mcp-Session-Id` header, that value is carried on every later request, and the open session is reused for the rest of the process. Replies are accepted either as plain JSON or as server-sent events. Successful results pass through verbatim, including any `isError` flag and any extra fields the service includes.

## Development

```bash
npm install
npm run build   # tsc -> dist/ (dist/cli.js is the bin)
npm test        # tsx --test test/*.test.ts — stubbed upstream, no network
npm run dev     # run the CLI from source
```

The test suite checks the served tool list against the captured surface (all 5 tools, deep-equal, in order, with no titles invented and no `required` key added to the four tools that carry none), exercises the session handshake and its reuse, verifies that no `Authorization` header is ever sent under any environment, and verifies that arguments go upstream untouched and results come back untouched.

## License

MIT — see `LICENSE`. Copyright (c) 2026 Rene Turcios.
