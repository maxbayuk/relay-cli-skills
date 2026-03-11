# Fee Estimation

Get a detailed fee breakdown for cross-chain operations without executing. Useful for budgeting, comparing costs at different amounts, and choosing optimal timing. **Read-only — does not execute.**

## Phase 1 — Parse Intent

Extract amount, token, origin, destination. Check if the user wants:
- **Single estimate** — one amount, one route
- **Comparison** — multiple amounts or routes side by side

## Phase 2 — Get Price Quote

Use the lightweight price endpoint for faster response:

```bash
relay price --params '{
  "user": "0x0000000000000000000000000000000000000001",
  "originChainId": 8453,
  "destinationChainId": 1,
  "originCurrency": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  "destinationCurrency": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "amount": "1000000000",
  "tradeType": "EXACT_INPUT"
}' --output json 2>/dev/null
```

**Tip:** Use a placeholder address (`0x...0001`) for price checks — no wallet needed for estimation.

For comparisons, run multiple quotes in parallel with different amounts.

## Phase 3 — Extract Fee Breakdown

| Field | Path | What It Is |
|-------|------|-----------|
| Output amount | `details.currencyOut.amountFormatted` | What the user receives |
| Relay fee | `fees.relayFee` | Protocol fee |
| Gas fee | `fees.gasFee` | Estimated gas cost |
| Total fee | Input amount - output amount | All-in cost |
| Price impact | `details.priceImpact` | Market impact (if swap) |

## Phase 4 — Present

**Single estimate:**

```
Fee Estimate: 1,000 USDC Base → Ethereum
Input:        1,000.00 USDC
Output:       ~998.50 USDC
Relay fee:    $0.80
Gas estimate: $0.70
Total cost:   ~$1.50 (0.15%)
Estimated time: ~5 seconds
```

**Comparison table:**

| Amount | Output | Fees | Fee % | Time |
|--------|--------|------|-------|------|
| 100 USDC | 98.20 | $1.80 | 1.80% | ~5s |
| 1,000 USDC | 998.50 | $1.50 | 0.15% | ~5s |
| 10,000 USDC | 9,988.00 | $12.00 | 0.12% | ~5s |

**Key insight:** Fees have a fixed + variable component. Larger amounts get proportionally cheaper.

## Fee Components

| Component | What It Is | Varies By |
|-----------|-----------|-----------|
| **Relay fee** | Protocol fee for facilitating the bridge | Route, token pair |
| **Gas fee** | Estimated gas for on-chain execution | Chain congestion, complexity |
| **Price impact** | Slippage from swapping in DEX pools | Amount vs pool depth |
| **Solver margin** | Spread taken by the solver (built into output) | Market conditions |

## Critical Rules

1. Use `price` not `quote` for estimation — it's faster and doesn't reserve liquidity.
2. Placeholder address is fine for price checks. No real wallet needed.
3. Fees change with market conditions — estimates are snapshots, not guarantees.
4. For swaps, always highlight price impact — it can dwarf bridge fees on large amounts.
