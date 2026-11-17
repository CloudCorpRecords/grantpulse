# BUILD REPORT — grantpulse

Clean-room rebuild of `com.dropwatchhq/research-intel`
v1.0.0 (remote-only MCP server at
`https://research.dropwatchhq.com/mcp`), built 2026-10-28
as a forwarding client. Package: `@reneturcios/grantpulse`
0.1.0, registry name `io.github.CloudCorpRecords/grantpulse`.

## Surface built

Full surface: **5 of 5 captured tools built**, none omitted.

| Tool | Required fields as served |
|------|---------------------------|
| `new_funding` | NO `required` key in the capture — served verbatim, none invented |
| `whats_changed_since` | NO `required` key in the capture — served verbatim, none invented |
| `delta_digest` | NO `required` key in the capture — served verbatim, none invented |
| `entity_watch` | `type`, `value` |
| `search_records` | NO `required` key in the capture — served verbatim, none invented |

Each served tool carries exactly the keys `name`, `description`,
`inputSchema`, exactly as captured: the capture carries NO `title`
keys, so none are served. The tool literals were generated
programmatically from
`harness/fixtures/researchintel-tools-original.json` by the script
at `/tmp/b48-grantpulse-scratch/gen.py` (parse + re-serialise; the
emitted `TOOL_DEFS` block was verified to byte-match the block in
`src/forward.ts` when stripped, and the generator's own round-trip
assertion confirmed the literals parse back to the fixture's tool
objects) — never retyped — and the registration-integrity test
suite plus the parity harness confirmed the served literals
deep-equal the fixture's 5 tool objects, in order. Tools are
registered via the MCP SDK low-level Server API
(`ListToolsRequestSchema` / `CallToolRequestSchema` handlers), not
McpServer+zod.

## Fixture conflicts

**None.** The spec and the fixture agree on all 5 tools' names,
descriptions, required fields (including the four tools with no
`required` key: `new_funding`, `whats_changed_since`,
`delta_digest`, `search_records`, and `entity_watch` requiring
`type` + `value`) and property shapes. The spec's note that the
capture carries no `title` keys matches the fixture exactly.

## Labelled reconstructions (from the spec's Wire section)

- Upstream endpoint pinned: `https://research.dropwatchhq.com/mcp`.
- Env override `RESEARCHINTEL_MCP_URL` — labelled reconstruction.
- The registry record declares NO headers on the remote: no auth
  header is ever sent. Tests assert that no `Authorization` header
  is sent under any environment — including with key-like env vars
  (`RESEARCHINTEL_API_KEY`, `API_KEY`, `AUTHORIZATION`) set — and
  that no `X-API-Key` header is ever sent; upstream requests carry
  only `Content-Type`, `Accept` and, when issued, `Mcp-Session-Id`.
- Error rendering `grantpulse error:` text for upstream failures —
  labelled reconstruction.
- Upstream results pass through verbatim, including `isError` and
  extra fields. tools/call is forwarded as streamable-HTTP
  JSON-RPC: initialize (protocolVersion `2025-03-26`) →
  notifications/initialized → tools/call, honouring
  `Mcp-Session-Id` when issued; plain-JSON and SSE replies are
  both parsed. The forwarding core takes an injectable fetch;
  tests stub the upstream — no live network calls in tests.

## Deviations

None.

## Lockfile

- Route: generated clean from birth in the build dir with no
  `node_modules` and no pre-existing lockfile:
  `npm install --package-lock-only --ignore-scripts --offline`
  (Route 3).
- Parses as JSON; `name` == `@reneturcios/grantpulse`.
- Package entries: **128**.
- sha256: `30b421c1e985456cd2801d9a85637df46abe2d5afc4bad87bd128c41638ad0ef`
  (re-verified unchanged after the subsequent offline `npm ci`).

## server.json

- Description exactly: "New NIH grants and clinical trials by institution, PI, sponsor, topic; funding-change digests." —
  **94 characters**.
- npm package entry carries `"version": "0.1.0"`; the OCI entry
  (`ghcr.io/cloudcorprecords/grantpulse:0.1.0`) carries **no**
  `version` field. Both entries carry packageArguments
  `[{type: positional, value: mcp}]` and transport stdio.
  Dockerfile (node:20-alpine, both stages) carries the label
  `io.modelcontextprotocol.server.name="io.github.CloudCorpRecords/grantpulse"`.
  Container workflow `IMAGE` is `ghcr.io/cloudcorprecords/grantpulse`
  and its smoke test asserts the server name `grantpulse`.

## Verification (all run 2026-10-28)

1. `npx tsc --noEmit` → exit 0.
2. `npm run build` → exit 0; real `dist/` produced (cli, forward,
   index, mcp, types — .js, .d.ts, .js.map each).
3. `node dist/cli.js --version` → `0.1.0`.
4. `npm test` → **55 tests, 55 pass, 0 fail** (registration-integrity
   suite deep-equals the served tools/list against the fixture's 5
   tools, including the no-title and required-key exactness
   assertions — entity_watch requires type+value, the other four
   have NO required key — plus session handling/reuse, env
   override, the no-Authorization-header assertions under any
   environment, verbatim argument forwarding and result
   pass-through including upstream isError, SSE parsing, error
   rendering, MCP-layer validation, and CLI behaviour including a
   tool subcommand forwarding test against a local stub server).
5. Parity: `python3 /home/hatch/workspace/mcp-rebuild-100/harness/parity.py --original /home/hatch/workspace/mcp-rebuild-100/harness/fixtures/researchintel-tools-original.json --rebuild "stdio:node /home/hatch/workspace/mcp-rebuild-100/builds/grantpulse/dist/cli.js mcp" --out /home/hatch/workspace/mcp-rebuild-100/builds/grantpulse/parity-out` → **verdict PASS, parity 100.0**
   (5/5 identical, 0 missing, 0 extra, no schema diffs). Output in
   `parity-out/`.
