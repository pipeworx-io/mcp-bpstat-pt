# mcp-bpstat-pt

BPstat — Banco de Portugal statistics API (Portuguese central bank). Keyless.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1683+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `list_domains` | List Banco de Portugal's statistical domains (the full subject tree). Each domain has id, parent_id (tree links; null = top level), label/short_label/description (Portuguese by default), num_series, num_datasets, and has_series. Start here, then use a domain id with list_datasets. lang defaults to PT; pass "EN" for English. |
| `list_datasets` | List the datasets within a statistical domain (use a domain id from list_domains). Returns a JSON-stat collection; each item has extension.id (the dataset id, a hex string) and extension.num_series. The dataset id feeds get_dataset. lang defaults to PT. |
| `get_dataset` | Fetch a dataset as JSON-stat — THIS is how you get actual data/observations. Returns the value[] and status[] arrays plus the dimension objects; observations align to dimension.reference_date (the time axis, ISO dates). extension.series[] lists every series (id + Portuguese label) the dataset contains. Requires both the parent domain_id and the dataset id (from list_datasets). Large datasets can be sizeable. lang defaults to PT. |
| `get_series_metadata` | Look up metadata for one or more series by numeric series id. Returns label, short_label, description (Portuguese by default), dataset_id, domain_ids, and dimension_category. Use this to identify a series and find which dataset_id holds its observations, then call get_dataset. NOTE: this endpoint returns metadata only — it does NOT return the numeric values. lang defaults to PT. |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "bpstat-pt": {
      "url": "https://gateway.pipeworx.io/bpstat-pt/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/bpstat-pt/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1683+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/list_domains \
  -H 'Content-Type: application/json' \
  -d '{"lang":"EN"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/list_domains`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "bpstat-pt": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-bpstat-pt"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-bpstat-pt
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Bpstat Pt data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
