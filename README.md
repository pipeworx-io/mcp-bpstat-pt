# mcp-bpstat-pt

BPstat — Banco de Portugal statistics API (Portuguese central bank). Keyless.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1394+ live data sources.

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

Or connect to the full Pipeworx gateway for access to all 1394+ data sources:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English:

```
ask_pipeworx({ question: "your question about Bpstat Pt data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
