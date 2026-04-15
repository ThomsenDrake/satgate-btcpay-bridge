# SatGate ↔ BTCPay Bridge

**Goal:** Demonstrate L402 payment authorization (SatGate) + settlement (BTCPay) for AI agent tool calls.

## Architecture

```
┌─────────────┐    L402 macaroon     ┌──────────────────┐   Invoice    ┌─────────────┐
│ AI Agent    │ ◄───────────────────► │ btcpay-mcp       │ ◄──────────► │ BTCPay      │
│ (MCP client)│                       │ (L402 validator) │              │ Server      │
└─────────────┘                       └────────▲─────────┘              └─────────────┘
                                                 │
                                    Budget check via macaroon caveat
                                                 │
                                       ┌──────────────────┐
                                       │ SatGate          │
                                       │ (authorization)  │
                                       └──────────────────┘
```

## Phase 1: L402 Validation Bridge

**Objective:** Issue a SatGate budget-check macaroon + validate it on BTCPay for a single MCP tool call.

### SatGate API (from source — v0.5.2)

SatGate exposes two API groups on port **8080**:

#### Capability APIs

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/capability/mint` | Mint a new capability token (admin only) |
| `POST` | `/api/capability/validate` | Validate a capability token |
| `POST` | `/api/capability/delegate` | Delegate a token with caveats |
| `GET` | `/api/capability/ping` | Health check with auth |

#### Governance APIs

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/governance/ban` | Ban/revoke a token |
| `GET` | `/api/governance/graph` | Token lineage graph |
| `POST` | `/api/governance/reset` | Reset governance data |

### Flow

1. **Mint** — SatGate admin mint a capability macaroon:
   ```bash
   curl -X POST http://localhost:8080/api/capability/mint \
     -H "Authorization: Bearer <admin_token>" \
     -H "Content-Type: application/json" \
     -d '{"scope": "btcpay-mcp", "budget": 1000}'
   ```
   Response: macaroon containing budget caveat in the mint response

2. **Validate** — btcpay-mcp validates before serving tools:
   ```bash
   curl -X POST http://localhost:8080/api/capability/validate \
     -H "Authorization: Bearer <macaroon>"
   ```

3. **Settlement** — SatGate proxies the MCP request; BTCPay creates invoice for Lightning settlement

### Implementation Status

- [x] Architecture documented
- [x] API endpoints identified (from SatGate source)
- [x] GitHub repo initialized
- [ ] SEP-1686 task POST format from Matt (SatGate)
- [ ] btcpay-mcp L402 middleware implementation
- [ ] End-to-end test

## Integration: tollbooth-dpyc (L402 + FastMCP)

The [tollbooth-dpyc](https://github.com/lonniev/tollbooth-dpyc) project demonstrates L402 + FastMCP session binding — a complementary pattern:

- **tollbooth-dpyc**: `session_id` bound to L402 macaroon via FastMCP
- **satgate-btcpay-bridge**: SatGate budget enforcement + BTCPay settlement
- **Combined**: SatGate authorizes → tollbooth-dpyc session-binds → BTCPay settles

This creates a full stack: authorization (SatGate) + session binding (tollbooth-dpyc/FastMCP) + settlement (BTCPay).

## Phase 2: Recurring Budget Monitoring

## Phase 3: Production Integration

## References

- [SatGate](https://satgate.io) — The Economic Firewall for AI Agents
- [btcpay-mcp](https://github.com/ThomsenDrake/btcpay-mcp) — BTCPay MCP server
- [tollbooth-dpyc](https://github.com/lonniev/tollbooth-dpyc) — L402 + FastMCP for MCP servers
- [x402 protocol](https://github.com/lightninglabs/x402) — L402 payment standard
- [SEP-1686](https://github.com/stacker-news/SEP-1686) — MCP Tasks standard
