# SEP-1686 Research — SatGate Task Tracker

**Date:** 2026-04-15
**Status:** CRITICAL FINDING — Architecture Rethink Required

## What SEP-1686 Actually Is

SEP-1686 is **NOT a REST API**. It is an **in-memory Go task tracker** inside the SatGate MCP proxy.

**Source:** `pkg/mcpserver/task_tracker.go` (satgate-io/satgate)

```go
type TaskTracker struct {
    mu    sync.RWMutex
    tasks map[string]*TaskSpend // taskID -> aggregated spend
}
```

**Key insight:** Task IDs are MCP protocol metadata — they flow through the JSON-RPC layer, NOT as HTTP POST calls.

## How It Works

1. When an MCP client sends a request, SatGate parses the `task_id` from JSON-RPC metadata
2. SatGate calls `taskTracker.RecordSpend(taskID, tokenID, budgetID, toolName, cost)` internally
3. BTCPay (btcpay-mcp) has NO direct API call to create tasks
4. The task correlation is metadata-only — btcpay-mcp would receive the `task_id` in the MCP protocol layer

## Correct Bridge Architecture

**Old approach (WRONG):**
- BTCPay calls `POST /api/v1/tasks` → SatGate
- Looking for a REST endpoint that doesn't exist

**Correct approach:**
- SatGate is the MCP proxy itself — it intercepts MCP traffic
- Task IDs come from the MCP protocol (`id` field in JSON-RPC requests)
- BTCPay as an MCP server would receive task_id in the MCP request metadata
- The "bridge" is really: SatGate (proxy) → BTCPay (MCP server) with task_id metadata flowing through

## What This Means for Phase 1

Phase 1 should be restructured:

1. **BTCPay as MCP server** receives requests via SatGate proxy
2. SatGate attaches `task_id` to each tool call (from JSON-RPC `id` field)
3. BTCPay logs/tracks spend against `task_id` (not SatGate calling BTCPay)
4. Budget check happens at SatGate proxy layer, not BTCPay layer

**The bridge is already the MCP protocol itself** — no new API needed.

## Action Items

- [ ] Update Matt on finding: SEP-1686 is MCP protocol task correlation, not HTTP API
- [ ] Propose revised Phase 1: BTCPay MCP server receives task_id metadata from SatGate proxy
- [ ] Close issue #37 (SEP-1686) on satgate repo with correct framing once Matt agrees
