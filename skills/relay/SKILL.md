---
name: relay
description: >-
  Cross-chain bridging and token swaps via Relay Protocol across 69+ chains.
  Handles quoting, execution, tracking, debugging, and portfolio management.
  Requires relay-cli: npx @relay-protocol/cli
allowed-tools: Bash(relay:*) Read
---

# Relay Protocol

Cross-chain bridging, swaps, and token management across 69+ EVM chains, Solana, and Bitcoin.

## Install

```bash
npm install -g @relay-protocol/cli
# or use without installing:
npx @relay-protocol/cli <command>
```

No API key needed for read operations (quotes, status, chains). Set `RELAY_API_KEY` for execution.

## Invocation

```bash
relay <command> --output json 2>/dev/null
```

Always use `--output json` and redirect stderr. Parse stdout as JSON.

## Workflows

Choose based on user intent:

| User wants to... | Reference |
|-------------------|-----------|
| Bridge tokens cross-chain | `references/workflow-bridge.md` |
| Swap tokens (same-chain or cross-chain) | `references/workflow-swap.md` |
| Compare routes and find best path | `references/workflow-best-route.md` |
| Execute multiple operations as a batch | `references/workflow-batch.md` |
| Estimate fees before committing | `references/workflow-fee-estimate.md` |
| Debug a failed or stuck transaction | `references/workflow-tx-debug.md` |
| Rebalance portfolio across chains | `references/workflow-portfolio-rebalance.md` |
| Optimize gas costs | `references/workflow-gas-optimize.md` |

## Critical Rules

1. **Amounts are always strings in wei.** `"1000000"` not `1000000`. Always strings, always smallest unit.
2. **Native token address is 40 zeros:** `0x0000000000000000000000000000000000000000`
3. **Non-standard chain IDs:** Bitcoin = 8253038, Solana = 792703809, Eclipse = 9286185
4. **Always `--output json 2>/dev/null`** — parse stdout as JSON, ignore stderr.
5. **Never execute without user confirmation.** Show the quote, get a "yes", then execute.
6. **Execute endpoints require `--confirm`.** Use `--dry-run` to preview first.
7. **Use `--hash` not `--txHash`** for transaction hash lookups.
8. **`intents/status` only takes requestId** — resolve hash via `relay requests list --hash 0x...` first.

## Token Decimals

| Token | Decimals | 1 unit in wei |
|-------|----------|---------------|
| USDC | 6 | 1000000 |
| USDT | 6 | 1000000 |
| ETH | 18 | 1000000000000000000 |

Convert human amounts: `amount_wei = amount * 10^decimals`

## Chain Shortcuts

`eth` (1), `base` (8453), `arb` (42161), `op` (10), `poly` (137), `bnb` (56), `avax` (43114), `sol` (792703809), `btc` (8253038)

Full chain list and token addresses: `references/chains.md`

## Field Filtering

Responses can be large (350KB). Use `--fields` (JMESPath) to compress:

```bash
relay chains list --fields "chains[].{id: id, name: name}"   # 350KB -> ~2KB
relay requests list --id 0x... --fields "requests[0].status"
```

## Error Handling

Route on error category, not message text. See `references/errors.md` for full catalog.

| Category | Action |
|----------|--------|
| Validation (`AMOUNT_TOO_LOW`, `INVALID_ADDRESS`) | Fix input, don't retry |
| API (`NO_QUOTES`, `UNSUPPORTED_ROUTE`) | Inform user, suggest alternative |
| Network / Rate limit / Server | Retry with exponential backoff |

## Schema Discovery

```bash
relay schema --list            # all endpoints
relay schema quote.v2          # params + response shape
```

## More Info

- Relay docs: https://docs.relay.link
- LLM-optimized docs: https://docs.relay.link/llms.txt
- CLI repo: https://github.com/relay-protocol/relay-cli
