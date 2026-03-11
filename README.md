# Relay Protocol Agent Skills

Cross-chain bridging and token swaps across 69+ chains — for AI agents.

## Install

Works with any [agentskills.io](https://agentskills.io)-compatible agent (Claude Code, Cursor, Codex, Gemini CLI, VS Code Copilot, Goose, and [32+ more](https://agentskills.io)):

```bash
npx skills add maxbayuk/relay-cli-skills
```

## What's Included

One skill (`relay`) with 8 workflow references:

| Workflow | Description |
|----------|-------------|
| **Bridge** | Move tokens between chains (quote → execute → track) |
| **Swap** | Same-chain or cross-chain token swaps |
| **Best Route** | Compare routes to find the cheapest/fastest path |
| **Batch** | Execute multiple operations with dependency tracking |
| **Fee Estimate** | Detailed fee breakdown before committing |
| **Tx Debug** | Diagnose failed or stuck transactions |
| **Portfolio Rebalance** | Redistribute tokens across chains to target allocations |
| **Gas Optimize** | Find the cheapest time and route for execution |

Plus shared references for chain IDs, token addresses, and error handling.

## Requires

- **relay-cli**: `npm install -g @relay-protocol/cli` or use `npx @relay-protocol/cli`
- No API key needed for read operations (quotes, status, chains)
- Set `RELAY_API_KEY` for execution

## Links

- [Relay Protocol](https://relay.link)
- [Relay Docs](https://docs.relay.link)
- [relay-cli on npm](https://www.npmjs.com/package/@relay-protocol/cli)
- [LLM-optimized docs](https://docs.relay.link/llms.txt)

## License

MIT
