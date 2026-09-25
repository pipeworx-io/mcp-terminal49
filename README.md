# mcp-terminal49

Terminal49 MCP — wraps the Terminal49 ocean/container shipment tracking API

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1679+ live data sources.

## Tools

| Tool | Description |
|------|-------------|
| `terminal49_track` | Start tracking an ocean shipment or container by submitting a bill of lading, booking, or container number plus the carrier SCAC. Returns the tracking request id and status (pending -> created/failed). Example: terminal49_track({ request_number: "MEDUFR030802", request_type: "bill_of_lading", scac: "MSCU", _apiKey: "your-key" }) |
| `terminal49_shipment` | Get an ocean shipment and its containers: bill of lading, carrier, port of lading (POL), port of discharge (POD), vessel, and ETA. Pass an `id` for a single shipment, or omit it to list recent shipments. Example: terminal49_shipment({ id: "3c9d...", _apiKey: "your-key" }) |
| `terminal49_container` | Get a container by id: current status, terminal holds, demurrage last free day (LFD / pickup_lfd), availability for pickup, and key milestone timestamps. Set include_events=true to also fetch normalized transport events (vessel/rail moves). Example: terminal49_container({ id: "8a1f...", include_events: true, _apiKey: "your-key" }) |

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "terminal49": {
      "url": "https://gateway.pipeworx.io/terminal49/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/terminal49/mcp` returns the tools in the table
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

Both URLs reach the same gateway and the same 1679+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

This pack takes your own API key (`_apiKey`) — we don't front one for it, so there's no curl here that would run without it. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/terminal49_track`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Standalone (no gateway account)

This package also runs as a local stdio MCP server — no Pipeworx account, no
gateway round-trip:

```json
{
  "mcpServers": {
    "terminal49": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-terminal49"]
    }
  }
}
```

Or run it directly to confirm it starts:

```bash
npx -y @pipeworx/mcp-terminal49
```

It speaks MCP over stdin/stdout and answers `initialize`/`tools/list`/`tools/call`
for **only** this pack's tools — none of the shared meta-tools the gateway
connection above adds. Same source, same tools, no ask_pipeworx routing.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Terminal49 data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
