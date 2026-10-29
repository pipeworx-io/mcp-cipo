# @pipeworx/cipo

Canadian patents and patent applications — search by owner or applicant name, or
look up one record by number for its status, filing date and issue date.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `search_cipo_patents(name, role, limit)` — search by current owner or
  applicant name and get back matching patents/applications: number, title,
  relevance. `role` is `"owner"` (current owner of record) or `"applicant"`
  (party that filed); these can differ after an assignment.
- `get_cipo_patent(number)` — one patent/application's full bibliographic
  record: English/French title, current status, filing date, issue/grant date
  (blank until granted), owners, applicants, inventors, IPC classification.

## Auth

Keyless. No registration, no API key.

## Data sources

- Canadian Intellectual Property Office (CIPO), Canadian Patents Database —
  every patent application filed in Canada and every patent granted by
  Canada, Canadian-origin or foreign-origin alike. Live per-request calls
  against CIPO's own public search service; nothing is downloaded or mirrored.

## Notes

- A patent that has only been filed, not yet granted, has an empty
  `issued_date` — that is CIPO's own data, not a missing field on our side.
- `filing_date` is whichever CIPO calls the earliest filing event for that
  record — the domestic filed date for most applications, or the PCT filing
  date for one that entered Canada via the PCT national phase. Both answer
  "when was this filed."
- Numbers are CIPO patent/application numbers, at most 7 digits. A "CA"
  prefix, spaces or dashes in the input are stripped automatically.
- CIPO's own CPD has had intermittent data-quality issues since its 2024
  "Next Generation Patents" migration (their notice, not ours) — status
  fields on some records may lag or show as blank even when the application
  is active. That is upstream, and this pack surfaces whatever CIPO currently
  reports.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "cipo": {
      "url": "https://gateway.pipeworx.io/cipo/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/cipo/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/search_cipo_patents \
  -H 'Content-Type: application/json' \
  -d '{"name":"Shopify Inc.","role":"applicant","limit":5}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/search_cipo_patents`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-cipo` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/cipo/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "cipo": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-cipo"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Cipo data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
