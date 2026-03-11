# Batch Cross-Chain Operations

Plan and execute multiple cross-chain operations as an optimized batch. Handles dependency ordering, parallel quoting, and sequential execution with tracking.

## Phase 1 — Decompose Into Operations

Break the user's request into individual operations. Each operation is one of:
- **bridge**: Move same token cross-chain
- **swap**: Exchange tokens (same or cross-chain)

Build a dependency graph:
```
Op 1: bridge 400 USDC ethereum → base         (independent)
Op 2: bridge 300 USDC ethereum → arbitrum      (independent)
Op 3: bridge 300 USDC ethereum → optimism      (independent)
```

Or with dependencies:
```
Op 1: bridge 2 ETH base → ethereum             (independent)
Op 2: swap ETH → USDC on ethereum              (depends on Op 1)
```

## Phase 2 — Quote All Operations

Quote independent operations in parallel:

```bash
# Run simultaneously
relay quote --params '{...op1...}' --output json 2>/dev/null &
relay quote --params '{...op2...}' --output json 2>/dev/null &
relay quote --params '{...op3...}' --output json 2>/dev/null &
wait
```

For dependent operations, quote sequentially — use the output amount from the previous step as the input amount for the next.

## Phase 3 — Present Execution Plan

Show the full plan before executing:

```
Batch Execution Plan (3 operations)
=====================================
[1] Bridge 400 USDC: Ethereum → Base       | Fee: $2.10 | ~5s
[2] Bridge 300 USDC: Ethereum → Arbitrum   | Fee: $1.80 | ~5s
[3] Bridge 300 USDC: Ethereum → Optimism   | Fee: $1.50 | ~5s
-------------------------------------
Total fees: $5.40
Operations [1-3] execute in parallel.
```

**Require explicit user confirmation before proceeding.**

## Phase 4 — Execute

Execute independent operations in parallel, dependent ones sequentially:

```bash
relay execute bridge --params '<op1_quote>' --confirm --output json 2>/dev/null &
relay execute bridge --params '<op2_quote>' --confirm --output json 2>/dev/null &
relay execute bridge --params '<op3_quote>' --confirm --output json 2>/dev/null &
wait
```

Collect request IDs from each execution.

## Phase 5 — Track All

Monitor all operations:

```bash
for id in $REQUEST_IDS; do
  relay status $id --output json 2>/dev/null
done
```

Report progress:
```
Batch Progress
[1] Bridge 400 USDC → Base:      Complete (tx: 0xabc...)
[2] Bridge 300 USDC → Arbitrum:  Pending
[3] Bridge 300 USDC → Optimism:  Complete (tx: 0xdef...)
```

## Common Batch Patterns

| Pattern | Description | Execution |
|---------|------------|-----------|
| Fan-out (1 → many) | Split tokens to multiple destinations | All parallel |
| Fan-in (many → 1) | Consolidate from multiple chains | All parallel |
| Sequential (chain) | Each step depends on previous | Sequential, re-quote each |
| Mixed | Combination of above | Build dependency graph |

## Critical Rules

1. Always show the full plan before executing — user must confirm the entire batch.
2. If any quote fails, show the partial plan and ask how to proceed.
3. For sequential operations, re-quote dependent steps after each execution — amounts may differ.
4. Track all operations to completion. Don't declare success until all are done or failed.
5. If one operation fails, report it but continue others (unless dependent).

## Limitations

- Maximum practical batch size: ~10 operations (quote expiry becomes a factor)
- The CLI doesn't have native batch execution — this workflow orchestrates individual calls
- Wallet must have sufficient balance for all parallel operations simultaneously
