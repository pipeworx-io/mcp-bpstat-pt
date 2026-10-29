# mcp-bpstat-pt

BPstat — Banco de Portugal statistics API (Portuguese central bank). Keyless.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

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

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/list_domains \
  -H 'Content-Type: application/json' \
  -d '{"lang":"EN"}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/list_domains`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-bpstat-pt` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/bpstat-pt/mcp` endpoint shown above. It needs network access;
there is no offline mode.

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
ask_pipeworx({ question: "your question about Bpstat Pt data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
