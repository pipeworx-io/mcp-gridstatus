# mcp-gridstatus

GridStatus MCP — wraps the GridStatus.io REST API (api.gridstatus.io/v1)

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1683+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `gridstatus_datasets` | List/search US grid datasets (LMP prices, load, fuel mix across the 7 ISOs: CAISO, ERCOT, PJM, MISO, SPP, NYISO, ISONE). Use this to discover the exact dataset_id to pass to gridstatus_query. Example: gridstatus_datasets({ filter: "caiso lmp", _apiKey: "your-key" }) |
| `gridstatus_query` | Query a grid dataset's time-series over a time window. Pass a dataset_id from gridstatus_datasets. Common ids: caiso_lmp_real_time_5_min, ercot_load, pjm_fuel_mix. Example: gridstatus_query({ dataset_id: "caiso_lmp_real_time_5_min", start_time: "2026-07-01T00:00Z", end_time: "2026-07-01T06:00Z", limit: 100, _apiKey: "your-key" }) |
| `gridstatus_latest` | Latest values for a grid dataset — convenience that pulls the most recent rows from a recent window. Pass a dataset_id (e.g. "ercot_load", "caiso_fuel_mix"). Example: gridstatus_latest({ dataset_id: "ercot_load", _apiKey: "your-key" }) |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "gridstatus": {
      "url": "https://gateway.pipeworx.io/gridstatus/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/gridstatus/mcp` returns the tools in the table
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

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/gridstatus_datasets`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "gridstatus": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-gridstatus"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-gridstatus
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Gridstatus data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
