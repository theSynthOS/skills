# Portfolio Monitoring & Yield Browsing

## Checking Portfolio

Use `portfolio.view` to see current positions and performance.

```
portfolio(walletAddress="0x...")
```

### What It Shows

- All active positions with protocol name, chain, and APY
- Deposited amount vs current redeemable amount (includes earned yield)
- Weighted average APY across all positions
- Wallet ETH balance (for gas context)

### When to Check Portfolio

- When user asks "how are my yields" or "show my positions"
- Before suggesting a rebalance (understand current state first)
- After a deposit/withdrawal to confirm it went through

## Browsing Available Yields

Use `yields.check` to explore what's available.

```
yields.check()
```

### Filtering

```
yields.check(token="USDC", minApy=0.05, chain="base", limit=5)
```

| Filter | Description | Example |
|--------|------------|---------|
| `token` | Token symbol | "USDC", "USDT" |
| `minApy` | Minimum APY (decimal) | 0.05 = 5% |
| `chain` | Chain name | "base", "arbitrum", "scroll" |
| `sortBy` | Sort field | "apy" (default), "name" |
| `limit` | Max results | 10 (default) |

### When to Browse vs Recommend

- **Browse** (`yields.check`): User wants to see what's available, compare options manually
- **Recommend** (`wallet.recommend`): User wants AI to pick the best allocation for them

If the user asks "what are the best yields?" — use `yields.check`.
If the user asks "where should I put my money?" — use `wallet.recommend`.

## Supported Chains

Bundie supports any EVM chain configured in the backend. Currently active:

| Chain | Chain ID | Common Protocols |
|-------|----------|-----------------|
| Scroll | 534352 | USX, various |
| Base | 8453 | Morpho, Moonwell |
| Arbitrum | 42161 | Various |
| Optimism | 10 | Moonwell |
| Mode | 34443 | Various |

New chains are added regularly — the MCP server dynamically passes `chainId` so new chains work automatically.

## Common Workflows

### "How am I doing?"
1. `portfolio.view` — show positions and APY
2. Summarize total value, weighted APY, and yield earned

### "What's the best yield right now?"
1. `yields.check(sortBy="apy", limit=5)` — top 5 by APY
2. Present with risk context ("highest APY but lower risk score")

### "Am I earning the best I can?"
1. `portfolio.view` — current positions
2. `portfolio.rebalance` — compare vs optimal
3. Present the delta and suggest action if significant
