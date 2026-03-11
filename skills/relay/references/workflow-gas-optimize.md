# Gas Optimization

Analyze and suggest the cheapest way to execute a cross-chain operation. Compares routing strategies and provides gas-saving recommendations. **Read-only — does not execute.**

## Phase 1 — Baseline Quote

Get the current cost for the straightforward route:

```bash
relay price --params '{
  "user": "0x0000000000000000000000000000000000000001",
  "originChainId": <origin>,
  "destinationChainId": <dest>,
  "originCurrency": "<token_origin>",
  "destinationCurrency": "<token_dest>",
  "amount": "<amount_wei>",
  "tradeType": "EXACT_INPUT"
}' --output json 2>/dev/null
```

## Phase 2 — Compare Alternative Routes

Test alternative strategies in parallel:

**Strategy A: Different intermediate token**
Some routes are cheaper if you bridge a different token and swap on the destination.

**Strategy B: Different origin chain**
If the user has the same token on multiple chains, one may be cheaper to bridge from.

**Strategy C: Split into smaller amounts**
Sometimes two smaller transfers cost less than one large one (different fee tiers).

Run 2-3 alternative quotes in parallel and compare total cost (fees + gas + price impact).

## Phase 3 — Chain Health Check

```bash
relay chains health --output json 2>/dev/null
```

Check for:
- Chains with elevated gas prices
- Degraded or slow chains (may have higher solver margins)
- Disabled chains (can't route through them)

## Phase 4 — Present Analysis

```
Gas Optimization Report: 10,000 USDC Ethereum → Base

Current best route:
  Direct USDC bridge: $3.20 total fees (0.032%)

Alternative routes tested:
  Bridge ETH + swap on Base: $8.50 (worse — swap adds cost)
  Split 5K + 5K:             $4.10 (worse — double fixed fees)

Recommendation: Direct USDC bridge at $3.20 is optimal.

Gas conditions:
  Ethereum: Normal (25 gwei)
  Base: Low (<0.01 gwei)

Tip: Ethereum gas is currently average. Bridging during
off-peak hours (weekends, early UTC morning) could save ~20-30%
on the gas component.
```

## Gas-Saving Heuristics

| Situation | Recommendation |
|-----------|---------------|
| Ethereum origin, > $10 gas fee | Consider waiting for lower gas |
| L2 → L2 transfer | Usually cheap regardless of timing |
| Large amount (> $100K) | Check price impact — may be cheaper to split |
| Recurring transfers | Batch into fewer, larger transfers |
| Token with deep liquidity (USDC, ETH) | Direct route almost always cheapest |
| Long-tail token | Swap to USDC → bridge → swap back may beat direct |

## Critical Rules

1. This is analysis only — does not execute.
2. Use `price` not `quote` for estimation — faster and doesn't reserve liquidity.
3. Placeholder address is fine for price checks.
4. Gas conditions change rapidly — always note that estimates are snapshots.
5. Don't over-optimize small amounts — if fees are < $1, the optimization isn't worth the complexity.

## Limitations

- Cannot predict future gas prices — can only report current conditions
- Some chains (Solana, Bitcoin) have different fee models not directly comparable
- Cannot compare with non-Relay bridges — only optimizes within Relay's routing
