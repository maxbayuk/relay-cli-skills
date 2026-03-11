# Token Swap

Swap tokens on the same chain or across chains. Relay automatically finds the best execution path — direct swap, bridge + swap, or multi-hop.

## Phase 1 — Parse Intent

Extract:
- **amount** — human-readable (convert to wei using token decimals)
- **tokenIn** — the token being sold (symbol or address)
- **tokenOut** — the token being bought (symbol or address)
- **originChain** — where tokenIn lives
- **destinationChain** — where tokenOut should arrive (same as origin for same-chain swaps)
- **user address** — sender wallet (ask if not provided)

**Same-chain vs cross-chain:** If origin and destination chains differ, Relay bridges + swaps in one transaction. The user doesn't need to specify — Relay handles routing.

## Phase 2 — Get Quote

**Same-chain swap:**
```bash
relay quote --params '{
  "user": "0x...",
  "originChainId": 8453,
  "destinationChainId": 8453,
  "originCurrency": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  "destinationCurrency": "0x0000000000000000000000000000000000000000",
  "amount": "100000000",
  "tradeType": "EXACT_INPUT"
}' --output json 2>/dev/null
```

**Cross-chain swap:**
```bash
relay quote --params '{
  "user": "0x...",
  "originChainId": 8453,
  "destinationChainId": 1,
  "originCurrency": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  "destinationCurrency": "0x0000000000000000000000000000000000000000",
  "amount": "100000000",
  "tradeType": "EXACT_INPUT"
}' --output json 2>/dev/null
```

**Present the quote:**
- Selling: {amount} {tokenIn} on {chain}
- Receiving: ~{output amount} {tokenOut} on {chain}
- Price impact: {impact}%
- Fees: {breakdown}

## Phase 3 — Preview

```bash
relay execute swap --params '<quote_response>' --dry-run --output json 2>/dev/null
```

## Phase 4 — Execute

**Require explicit user confirmation.**

```bash
relay execute swap --params '<quote_response>' --confirm --output json 2>/dev/null
```

## Phase 5 — Track

```bash
relay status <requestId> --output json 2>/dev/null
```

## EXACT_INPUT vs EXACT_OUTPUT

| Trade Type | Use When | Amount Field Means |
|-----------|----------|-------------------|
| `EXACT_INPUT` | "I want to sell exactly 100 USDC" | Exact amount being sold |
| `EXACT_OUTPUT` | "I want to receive exactly 0.05 ETH" | Exact amount to receive |

Default to `EXACT_INPUT` unless the user specifies a desired output amount.

## Swap-Specific Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `NO_QUOTES` | No liquidity path for this pair | Try a different token pair or larger amount |
| `PRICE_IMPACT_TOO_HIGH` | Swap would move the market significantly | Reduce amount or split into smaller swaps |
| `INSUFFICIENT_LIQUIDITY` | Pool doesn't have enough tokens | Reduce amount |

## Critical Rules

1. Same originCurrency and destinationCurrency is invalid — can't swap a token for itself.
2. Always show the quote before executing — users must confirm the output amount and price impact.
3. Quotes expire in ~30 seconds — don't delay between quote and execute.
