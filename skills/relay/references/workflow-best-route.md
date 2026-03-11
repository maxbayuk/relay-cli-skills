# Best Route Finder

Compare routes to find the optimal path for a cross-chain transfer. Evaluates cost, speed, and output amount across different strategies. **Read-only — does not execute.**

## Phase 1 — Parse Intent

Extract amount, token, origin chain, destination chain, and user address.

Determine what the user optimizes for:
- **Cost** (default) — maximize output amount, minimize fees
- **Speed** — fastest fill time
- **Reliability** — most likely to succeed

## Phase 2 — Generate Route Candidates

Request quotes for multiple strategies in parallel:

**Strategy 1: Direct bridge (same token)**
```bash
relay quote --params '{
  "user": "0x...",
  "originChainId": <origin>,
  "destinationChainId": <dest>,
  "originCurrency": "<token_address_origin>",
  "destinationCurrency": "<token_address_dest>",
  "amount": "<amount_wei>",
  "tradeType": "EXACT_INPUT"
}' --output json 2>/dev/null
```

**Strategy 2: Bridge native + swap on destination**
```bash
relay quote --params '{
  "originCurrency": "0x0000000000000000000000000000000000000000",
  "destinationCurrency": "<target_token_dest>",
  ...
}' --output json 2>/dev/null
```

**Strategy 3: Swap on origin + bridge**
```bash
relay quote --params '{
  "originCurrency": "<different_token_origin>",
  "destinationCurrency": "<target_token_dest>",
  ...
}' --output json 2>/dev/null
```

Run all three in parallel (separate bash commands).

## Phase 3 — Compare Routes

Build a comparison table:

| Route | Output Amount | Fees | Est. Time | Steps |
|-------|-------------|------|-----------|-------|
| Direct USDC bridge | 9,985.50 USDC | $14.50 | ~5s | 1 |
| ETH bridge + swap | 9,978.20 USDC | $21.80 | ~15s | 2 |
| Swap to ETH + bridge | 9,970.00 USDC | $30.00 | ~20s | 2 |

Extract from each quote response:
- `details.currencyOut.amountFormatted` — output amount
- `fees` — fee breakdown
- `details.timeEstimate` — estimated fill time
- `steps` — number of execution steps

## Phase 4 — Recommend

Present the comparison and recommend the best route based on the user's priority. Hand off to `workflow-bridge.md` or `workflow-swap.md` to execute.

## When to Suggest Multi-Hop

Multi-hop (swap + bridge) can beat direct bridging when:
- The direct route has low liquidity (high price impact)
- A different token has a deeper liquidity pool on the destination
- The user is bridging a long-tail token not directly supported

## Critical Rules

1. This workflow is read-only — it quotes but does not execute.
2. Run quotes in parallel to minimize latency.
3. Quotes expire quickly — if the user decides to execute, they should do so immediately.
4. Some strategies may return `NO_QUOTES` — that's fine, just exclude from comparison.
