# Cross-Chain Bridge

Move tokens between chains using Relay Protocol's instant bridging. Handles quoting, execution, and tracking.

## Phase 1 — Parse Intent

Extract from the user's request:
- **amount** — in human-readable units (e.g., "100 USDC" = 100000000 in wei for 6-decimal token)
- **token** — symbol (USDC, USDT, ETH) or contract address
- **origin chain** — name or chain ID
- **destination chain** — name or chain ID
- **user address** — the sender's wallet address (ask if not provided)
- **recipient** — defaults to user address if not specified

Convert human amounts to wei: `amount_wei = amount * 10^decimals`

## Phase 2 — Get Quote

```bash
relay bridge --from <origin> --to <dest> --token <token> --amount <amount_wei> --user <address> --output json 2>/dev/null
```

Or with raw params for full control:

```bash
relay quote --params '{
  "user": "0x...",
  "originChainId": 8453,
  "destinationChainId": 1,
  "originCurrency": "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
  "destinationCurrency": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "amount": "100000000",
  "tradeType": "EXACT_INPUT"
}' --output json 2>/dev/null
```

**Present the quote to the user:**
- Origin: {amount} {token} on {chain}
- Destination: {output amount} {token} on {chain}
- Fees: {relay fee} + {gas estimate}
- Estimated time: {time}

Use `--fields` to extract key fields if the response is large:
```bash
--fields "{fees: fees, details: details.currencyOut}"
```

## Phase 3 — Preview (Dry Run)

Before executing, always preview:

```bash
relay execute bridge --params '<quote_response>' --dry-run --output json 2>/dev/null
```

This prints the equivalent curl command without executing. Show the user:
- Exact transaction that will be submitted
- Contract being called
- Value being sent

## Phase 4 — Execute

**Require explicit user confirmation before this step.**

```bash
relay execute bridge --params '<quote_response>' --confirm --output json 2>/dev/null
```

Present the result:
- Transaction hash
- Request ID (for tracking)
- Origin chain explorer link

## Phase 5 — Track

Poll until complete:

```bash
relay status <requestId> --output json 2>/dev/null
```

Or with full details:

```bash
relay requests list --id <requestId> --fields "requests[0].{status: status, fillTx: fillTxHashes, origin: inTxHashes}" --output json 2>/dev/null
```

**Status values:** `pending` → `waiting` → `delayed` → `success` or `failure`

If status is `failure`, check the fill transaction on-chain for revert reason.

## Error Handling

| Error | Category | Action |
|-------|----------|--------|
| `AMOUNT_TOO_LOW` | validation | Increase amount — minimum varies per route |
| `NO_QUOTES` | api | Route not supported — try different token pair or chain |
| `UNSUPPORTED_CURRENCY` | api | Token not available on this chain via Relay |
| 429 / rate limit | rate_limit | Wait 5s, retry with backoff |
| 5xx | server | Retry up to 3 times with exponential backoff (2s→30s) |
| Network timeout | network | Retry up to 5 times with exponential backoff (1s→30s) |

## Limitations

- EVM chains + Solana + Bitcoin — check `relay chains list` for supported chains
- Quotes expire — execute within ~30 seconds of receiving a quote
- Execute requires wallet signing — the CLI returns unsigned transaction payloads; the calling agent/wallet must sign and submit
- Not all token pairs are routable — some combinations have no liquidity path
