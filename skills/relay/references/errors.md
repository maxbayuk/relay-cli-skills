# Error Handling Reference

## Error Categories

Errors from relay-cli fall into these categories. Route on category, not message text.

### Validation Errors (Do Not Retry)

| Error Code | Cause | Fix |
|-----------|-------|-----|
| `AMOUNT_TOO_LOW` | Amount below minimum for this route | Increase amount |
| `AMOUNT_TOO_HIGH` | Amount exceeds available liquidity | Decrease amount or split |
| `INVALID_ADDRESS` | Malformed wallet address | Check 0x prefix + 40 hex chars |
| `INVALID_CHAIN_ID` | Unrecognized chain ID | Use `relay chains list` to verify |
| `MISSING_REQUIRED_PARAM` | Required field missing from request | Check `relay schema <endpoint>` |

### API Errors (Do Not Retry Same Request)

| Error Code | Cause | Fix |
|-----------|-------|-----|
| `NO_QUOTES` | No route available for this pair | Try different token, chain, or amount |
| `UNSUPPORTED_CURRENCY` | Token not available via Relay on this chain | Check supported tokens |
| `UNSUPPORTED_ROUTE` | Chain pair not bridgeable | Verify chains are supported |
| `QUOTE_EXPIRED` | Quote TTL exceeded | Request a fresh quote |

### Network Errors (Retryable)

| Error | Strategy |
|-------|----------|
| DNS failure | Exponential backoff: 1s → 2s → 4s → 8s → 16s (max 5 attempts) |
| Connection timeout | Same as DNS failure |
| Socket hang up | Same as DNS failure |

### Rate Limit (Retryable)

| Error | Strategy |
|-------|----------|
| HTTP 429 | Fixed 5s delay, then exponential backoff |

### Server Errors (Retryable)

| Error | Strategy |
|-------|----------|
| HTTP 500-599 | Exponential backoff: 2s → 4s → 8s (max 3 attempts) |

## Error Response Shape

```json
{
  "error": {
    "code": "AMOUNT_TOO_LOW",
    "message": "Amount is below the minimum for this route",
    "details": { "minimum": "50000" }
  }
}
```

## Decision Table

```
Error received
├── Is it a validation error? → Fix the input, do not retry
├── Is it an API error? → Inform user, suggest alternative
├── Is it a network/rate/server error? → Retry with backoff
└── Unknown error → Log full response, inform user
```
