# Supported Chains & Shortcuts

## Chain Name Shortcuts

The CLI resolves human-friendly names to chain IDs:

| Shortcut | Chain | Chain ID |
|----------|-------|----------|
| `eth` | Ethereum | 1 |
| `base` | Base | 8453 |
| `arb` | Arbitrum | 42161 |
| `op` | Optimism | 10 |
| `poly`, `matic` | Polygon | 137 |
| `bnb` | BNB Chain | 56 |
| `avax` | Avalanche | 43114 |
| `sol` | Solana | 792703809 |
| `btc` | Bitcoin | 8253038 |
| `zora` | Zora | 7777777 |
| `blast` | Blast | 81457 |
| `linea` | Linea | 59144 |
| `scroll` | Scroll | 534352 |
| `zksync` | zkSync Era | 324 |

## Non-Standard Chain IDs

These are NOT EVM chain IDs — they're Relay-internal identifiers:

| Chain | Relay Chain ID | Notes |
|-------|---------------|-------|
| Bitcoin | 8253038 | BTC native only |
| Solana | 792703809 | SPL tokens supported |
| Eclipse | 9286185 | SVM-based L2 |

## Common Token Addresses

### USDC
| Chain | Address |
|-------|---------|
| Ethereum (1) | `0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48` |
| Base (8453) | `0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913` |
| Arbitrum (42161) | `0xaf88d065e77c8cC2239327C5EDb3A432268e5831` |
| Optimism (10) | `0x0b2C639c533813f4Aa9D7837CAf62653d097Ff85` |
| Polygon (137) | `0x3c499c542cEF5E3811e1192ce70d8cC03d5c3359` |

### USDT
| Chain | Address |
|-------|---------|
| Ethereum (1) | `0xdAC17F958D2ee523a2206206994597C13D831ec7` |
| Arbitrum (42161) | `0xFd086bC7CD5C481DCC9C85ebE478A1C0b69FCbb9` |
| Optimism (10) | `0x94b008aA00579c1307B0EF2c499aD98a8ce58e58` |

### Native Token (ETH / gas token)
All chains: `0x0000000000000000000000000000000000000000`

## Discovering Chains at Runtime

```bash
# List all supported chains
relay chains list --fields "chains[].{id: id, name: name, displayName: displayName}" --output json

# Slim preset (smaller response)
relay chains list --preset slim --output json

# Check chain health / disabled status
relay chains health --fields "chains[?disabled==\`true\`].{id: id, name: name}" --output json
```
