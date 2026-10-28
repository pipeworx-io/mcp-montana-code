# Montana Code Annotated

Montana state statutes by citation, plus topic search across the whole code.
Keyless.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `mt_code_section` -- the full text of one section by citation, e.g.
  `1-1-307` (postponement of a day appointed for an action when it falls on a
  holiday or Saturday) or `45-5-102` (deliberate homicide).
- `mt_code_search` -- full-text search by topic or keyword, e.g.
  "computing time period days holiday". Returns ranked sections with their
  citation, heading and an opening excerpt; call `mt_code_section` with a
  result for the full text. A query with no hits returns `found: false`, never
  an empty list.

## Coverage

All 54 titles of the Montana Code Annotated (2025 edition), about 44,000
sections. That includes Title 25, chapter 20, the Montana Rules of Civil
Procedure (Rule 6, computing and extending time, is MCA 25-20-206), and the
other court rule sets published in Title 25.

Search is a full-text match, not semantic search: it ranks sections containing
all the query's words, and falls back to any-word matching when no section
contains them all.

## How search works

`mt_code_search` uses the shared statute search store: an FTS5 index in the
`SEARCH_SHARD` Durable Object (shard name `mt-statutes`) and the section text
in the shared datasets R2 bucket (`statutes/mt/<title>.json`, one bundle per
title, plus `_index.json`). No new Cloudflare binding. The index is loaded by
`scripts/ingest-montana-statutes.mjs`; re-run it after the Legislature
publishes a new edition. Every response carries `data_as_of`.

## Data source

Montana Legislature, Montana Code Annotated. Montana statutes are public
record. Citation lookups call the Legislature live on each request; topic
search reads a pre-fetched copy of the current code.

## Stored copy and `data_as_of` (fleet #3754)

`mt_code_section` answers from the Montana Code Annotated already held in the shared statute store (R2,
`statutes/mt/`, one object per title, loaded by `scripts/ingest-montana-statutes.mjs` for topic search). The
legislature's archive site refuses requests that leave from some data centres (70 "upstream down" against 171
successes from external callers over 14 days). A section the stored title lists is answered with no upstream
request: `served_from: "stored-copy"`, `data_as_of` (when the title was last read from the source) and the
section's `history` line. A title that cannot be read, or a section it does not list, falls back to the live read.
Refreshing the store refreshes both search and lookup.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "montana-code": {
      "url": "https://gateway.pipeworx.io/montana-code/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/montana-code/mcp` returns the tools in the table
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
curl -X POST https://gateway.pipeworx.io/v1/tools/mt_code_section \
  -H 'Content-Type: application/json' \
  -d '{"section":"45-5-102"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/mt_code_section`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "montana-code": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-montana-code"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-montana-code
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Montana Code data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
