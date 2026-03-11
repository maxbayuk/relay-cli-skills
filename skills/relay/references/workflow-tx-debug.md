# Transaction Debugging

Diagnose failed, stuck, or unexpected Relay transactions. Traces through the request lifecycle to identify the failure point.

## Phase 1 — Identify the Request

**If given a request ID:**
```bash
relay requests list --id <requestId> --output json 2>/dev/null
```

**If given a transaction hash:**
```bash
relay requests list --hash <txHash> --output json 2>/dev/null
```

**Important:** Use `--hash`, not `--txHash` or `--inTxHash` — those return unrelated results.

Extract from the response:
- `requests[0].status` — current state
- `requests[0].data.metadata` — request metadata (note: nested under `data`)
- `requests[0].inTxHashes` — origin transaction(s)
- `requests[0].fillTxHashes` — fill transaction(s) on destination

## Phase 2 — Diagnose by Status

| Status | Meaning | Next Step |
|--------|---------|-----------|
| `pending` | Request received, not yet picked up by solver | Check if recently submitted — may just need time |
| `waiting` | Solver accepted, waiting for on-chain confirmation | Check origin chain tx status |
| `delayed` | Taking longer than expected | Check chain congestion, solver capacity |
| `success` | Completed successfully | Verify output amount matches expected |
| `failure` | Failed | → Phase 3 (failure analysis) |
| `refunded` | Funds returned to sender | Verify refund tx on origin chain |

## Phase 3 — Failure Analysis

For failed requests, check the fill transaction on the destination chain:

```bash
relay requests list --id <requestId> --fields "requests[0].{status: status, fillTx: fillTxHashes, inTx: inTxHashes, origin: originChainId, dest: destinationChainId}" --output json 2>/dev/null
```

**Common failure patterns:**

| Pattern | Cause | Resolution |
|---------|-------|------------|
| No fill tx hash | Solver never attempted to fill | Route may have been unprofitable — retry with fresh quote |
| Fill tx reverted | On-chain execution failed | Check revert reason (slippage, liquidity, gas) |
| Origin tx failed | User's deposit tx reverted | Check approval, balance, gas |
| Stuck in `pending` > 5 min | No solver picked it up | Quote may have expired — get a new quote |

## Phase 4 — Check Chain State

If the origin or fill transaction exists, verify on-chain:

```bash
relay chains list --fields "chains[?id==\`<chainId>\`].{name: name, explorers: explorerUrls}" --output json 2>/dev/null
```

Provide the user with:
- Block explorer links for both origin and destination transactions
- The specific revert reason if available
- Whether funds are safe (still in user's wallet, in the bridge contract, or on destination)

## Phase 5 — Report

Present a structured diagnosis:

```
Transaction Debug Report
Request ID: 0x67015eb...
Status: failure

Origin:      1,000 USDC on Base (tx: 0xabc...)  Confirmed
Fill:        Failed on Ethereum (tx: 0xdef...)   Reverted

Failure: Fill transaction reverted — insufficient liquidity in swap pool.

Funds status: Deposited in bridge contract. Eligible for refund.

Recommended action: Wait for automatic refund (usually < 30 minutes),
or retry with a fresh quote for a smaller amount.
```

## Stuck Transaction Checklist

If a transaction is stuck (not failed, just slow):

1. **Check chain health:**
   ```bash
   relay chains health --output json 2>/dev/null
   ```
   Look for disabled chains or degraded status.

2. **Check request age:** If < 2 minutes, likely still processing normally.
3. **Check solver activity:** High solver queue times can delay fills.
4. **Advise patience or retry:** Most stuck transactions resolve within 10 minutes. If > 30 minutes, likely needs manual intervention.

## Critical Rules

1. Use `--hash` not `--txHash` for transaction hash lookups.
2. Metadata is nested: `requests[0].data.metadata`, NOT `requests[0].metadata`.
3. Never suggest the user manually interact with bridge contracts — if funds are stuck, they should contact Relay support.
4. Always report fund safety — the first thing users want to know is "are my funds safe?"
