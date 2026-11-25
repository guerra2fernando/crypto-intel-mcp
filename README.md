# crypto-intel-mcp

An MCP (Model Context Protocol) server exposing on-chain **crypto intelligence** for Base tokens, with an x402-paid endpoint for the expensive on-chain reads.

- **MCP endpoint (JSON-RPC 2.0 over HTTP):** https://crypto-intel-mcp.lengrowth.workers.dev/mcp
- **x402 paid REST endpoint:** https://crypto-intel-mcp.lengrowth.workers.dev/paid
- **Transport:** Streamable HTTP, `POST` JSON-RPC 2.0
- **Network:** Base mainnet (eip155:8453)

## Tools

### `token_risk`

Risk / liquidity / momentum report for a Base token.

Input schema (`tools/list` -> `inputSchema`):

```json
{
  "type": "object",
  "properties": {
    "token": { "type": "string", "description": "Base token 0x address" },
    "tx":    { "type": "string", "description": "Base tx hash of USDC payment to release the paid report" }
  },
  "required": ["token"]
}
```

Returns liquidity, holder concentration, momentum and a composite risk score for the requested Base token.

## Pricing (x402)

`token_risk` is paid. The first call returns an **HTTP 402** challenge describing the payment. Pay it on Base, then re-call with `tx` set to the payment transaction hash.

| Field | Value |
|---|---|
| Protocol | x402 v1 |
| Network | Base (`eip155:8453`) |
| Asset | USDC on Base `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` |
| `payTo` | `0x6bf8ac13aaf4c928b75998cf06ce256e59936400` |
| Price | `10000` atomic units = **0.01 USDC** |
| Endpoint | `POST /paid` |

Example challenge:

```bash
curl -s -X POST https://crypto-intel-mcp.lengrowth.workers.dev/mcp \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"token_risk","arguments":{"token":"0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913"}}}'
```

## Usage

List tools:

```bash
curl -s -X POST https://crypto-intel-mcp.lengrowth.workers.dev/mcp \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

Call a tool:

```bash
curl -s -X POST https://crypto-intel-mcp.lengrowth.workers.dev/mcp \
  -H 'content-type: application/json' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"token_risk","arguments":{"token":"0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913"}}}'
```

### MCP client config

```json
{
  "mcpServers": {
    "crypto-intel": {
      "type": "http",
      "url": "https://crypto-intel-mcp.lengrowth.workers.dev/mcp"
    }
  }
}
```

## Source

- `workers/crypto-intel-mcp.js` — the MCP-over-HTTP worker (JSON-RPC dispatch, `tools/list`, `tools/call`, x402 challenge/verify).

## License

MIT
