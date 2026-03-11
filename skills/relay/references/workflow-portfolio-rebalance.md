# Portfolio Rebalance

Plan and execute multi-chain portfolio rebalancing. Calculates the minimal set of transfers needed to reach target allocations.

## Phase 1 — Assess Current Holdings

The user must provide their wallet address and the token to rebalance. Relay CLI doesn't have a native balance checker, so the agent/user must supply current balances.

**If balances are provided:**
Build the current allocation map:
```
Current USDC holdings:
  Ethereum:  5,000 USDC
  Base:      3,000 USDC
  Arbitrum:  2,000 USDC
  Total:     10,000 USDC
```

**If balances are NOT provided:**
Ask the user for their current balances per chain, or suggest they use a portfolio tracker.

## Phase 2 — Calculate Target Allocation

**Consolidation (move everything to one chain):**
```
Target: 100% Ethereum
  Ethereum: 10,000 USDC (need +5,000)
  Base:     0 USDC (need -3,000)
  Arbitrum: 0 USDC (need -2,000)
```

**Percentage-based allocation:**
```
Target: 50% / 30% / 20%
  Ethereum: 5,000 USDC (no change)
  Base:     3,000 USDC (no change)
  Arbitrum: 2,000 USDC (no change)
  Already balanced! No transfers needed.
```

Calculate the **delta** for each chain: `target - current`.

## Phase 3 — Generate Transfer Plan

Convert deltas into the minimal set of transfers:
- Chains with negative delta = sources
- Chains with positive delta = destinations
- Match sources to destinations to minimize number of transfers

## Phase 4 — Quote All Transfers

Quote each transfer in parallel:

```bash
relay price --params '{
  "user": "0x...",
  "originChainId": 8453,
  "destinationChainId": 1,
  "originCurrency": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  "destinationCurrency": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "amount": "3000000000",
  "tradeType": "EXACT_INPUT"
}' --output json 2>/dev/null
```

## Phase 5 — Present Rebalance Plan

```
Rebalance Plan: Consolidate USDC to Ethereum

Current                     Target
  ETH:  5,000 USDC           ETH:  10,000 USDC
  Base: 3,000 USDC     →     Base: 0 USDC
  Arb:  2,000 USDC           Arb:  0 USDC

Transfers:
[1] 3,000 USDC Base → Ethereum    | Fee: ~$1.50 | ~5s
[2] 2,000 USDC Arbitrum → Ethereum | Fee: ~$1.20 | ~5s
                                     Total fees: ~$2.70
                            Final balance: ~9,997.30 USDC on Ethereum

Execute in parallel (no dependencies).
```

**Require explicit user confirmation.**

## Phase 6 — Execute and Track

Hand off to the batch workflow pattern — execute independent transfers in parallel, track all to completion. See `workflow-batch.md`.

## Optimization Tips

1. **Minimize transfers:** If current allocation is close to target, skip small moves where fees would eat into the amount.
2. **Fee threshold:** Suggest skipping transfers where fees > 1% of the amount being moved.
3. **Timing:** If no urgency, suggest waiting for lower gas periods.

## Critical Rules

1. Relay CLI doesn't check balances — the user or upstream agent must provide current holdings.
2. Always show the full plan before executing.
3. Account for fees in the target — the received amount will be slightly less than the sent amount.
4. Never assume token addresses — verify with `relay chains list` or the chains reference.

## Limitations

- Cannot query on-chain balances directly (user must provide)
- Fee estimates are snapshots — actual fees may differ at execution
- Maximum practical parallel transfers: ~10 (wallet nonce management)
