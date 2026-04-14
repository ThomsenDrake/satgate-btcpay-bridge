# SatGate ↔ BTCPay Bridge

**Goal:** Demonstrate L402 payment authorization (SatGate) + settlement (BTCPay) for AI agent tool calls.

## Architecture

```
┌─────────────┐    L402 macaroon     ┌──────────────────┐   Invoice    ┌─────────────┐
│ AI Agent    │ ◄───────────────────► │ btcpay-mcp       │ ◄──────────► │ BTCPay      │
│ (MCP client)│                       │ (L402 validator) │              │ Server      │
└─────────────┘                       └────────▲─────────┘              └─────────────┘
                                                 │
                                    Budget check │ SEP-1686
                                                 ▼
                                       ┌──────────────────┐
                                       │ SatGate          │
                                       │ (authorization)  │
                                       └──────────────────┘
```

## Phase 1: L402 Validation Bridge

**Objective:** Issue a SatGate budget-check macaroon + validate it on BTCPay for a single MCP tool call.

### Integration Points

1. **SatGate side** — Issue macaroon on budget check:
   - Endpoint: `POST /api/v1/authorize`
   - Request: `{ "task_id": "...", "budget": "...", "client": "btcpay-mcp" }`
   - Response: macaroon + L402 payment request

2. **BTCPay side** — Validate L402 payment:
   - `L402-Token` header with SatGate macaroon
   - BTCPay `ValidateMacaroon()` middleware
   - If valid → serve MCP tool, create invoice for settlement

3. **Settlement** — Invoice lifecycle:
   - BTCPay creates invoice on tool call
   - SatGate pays via pre-funded Lightning channel
   - Completion webhook fires → release MCP tool response

### Unknowns (need SatGate input)
- [ ] SEP-1686 task creation endpoint (POST format, required fields)
- [ ] Budget enforcement webhook URL or GET endpoint
- [ ] Test credentials / sandbox environment

## Phase 2: Recurring Budget Monitoring

## Phase 3: Production Integration

## References

- [SatGate](https://satgate.io)
- [btcpay-mcp](https://github.com/ThomsenDrake/btcpay-mcp)
- [x402 protocol](https://github.com/lightninglabs/x402)
- [SEP-1686](https://github.com/stacker-news/SEP-1686)
