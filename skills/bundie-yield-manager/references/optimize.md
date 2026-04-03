# Recommendations, Rebalancing & Preferences

## Getting AI Recommendations

Use `get_recommendation` to get a personalized yield bundle.

```
get_recommendation(walletAddress="0x...")
```

### How It Works

1. Analyzes the wallet (or uses cached analysis)
2. Runs a **bull/bear/moderator AI debate** to select optimal strategies
3. Returns a diversified bundle with allocation percentages

### Customizing Recommendations

Pass optional filters:
```
get_recommendation(
  walletAddress="0x...",
  tokens=["USDC"],
  minRiskScore=80,
  bundleSize=3
)
```

Or set persistent preferences first:
```
set_preferences(
  walletAddress="0x...",
  minRiskScore=80,
  auditedOnly=true,
  maxAllocationPerProtocol=30
)
```

Then `get_recommendation` will use those preferences automatically.

### Acting on Recommendations

After the user approves a recommendation:
1. `deposit` — deposit the total amount to vault
2. For each item in the bundle, `strategy_deposit` with the allocated amount

## Rebalancing

Use `rebalance` to compare current positions vs optimal allocation.

```
rebalance(walletAddress="0x...")
```

### Auto-Execute

```
rebalance(walletAddress="0x...", autoExecute=true)
```

This will reallocate funds based on the suggestions. Only use if the user explicitly agrees.

### When to Suggest Rebalancing

- When yields have shifted significantly since last allocation
- When a user asks "should I rebalance" or "is my portfolio optimal"
- Periodically (e.g., weekly check-ins)

## Migration Opportunities

Use `migrate` to find better yield vs external DeFi positions.

```
migrate(walletAddress="0x...")
```

Requires a prior wallet analysis. Shows:
- Current external positions with APY
- Better Bundie alternatives
- APY improvement delta
- Idle assets that could be earning

## Setting Preferences

Use `set_preferences` to establish rules for the session.

```
set_preferences(
  walletAddress="0x...",
  maxAllocationPerProtocol=30,
  auditedOnly=true,
  minRiskScore=75,
  preferredTokens=["USDC"],
  excludeProtocols=["risky-protocol"]
)
```

### Available Preferences

| Preference | Type | Description |
|-----------|------|-------------|
| `maxAllocationPerProtocol` | number | Max % in any single protocol |
| `auditedOnly` | boolean | Only audited protocols |
| `minRiskScore` | number | Minimum risk score (0-100) |
| `maxRiskScore` | number | Maximum risk score (0-100) |
| `excludeChains` | number[] | Chain IDs to exclude |
| `excludeProtocols` | string[] | Protocol names to exclude |
| `preferredTokens` | string[] | Preferred tokens |

Preferences persist for the current session and are applied to all subsequent recommendations and rebalance checks.

### Natural Language Mapping

| User says | Preference |
|-----------|-----------|
| "only audited protocols" | `auditedOnly: true` |
| "max 30% in one place" | `maxAllocationPerProtocol: 30` |
| "nothing below B+ risk" | `minRiskScore: 75` |
| "only USDC" | `preferredTokens: ["USDC"]` |
| "no Arbitrum" | `excludeChains: [42161]` |
| "avoid protocol X" | `excludeProtocols: ["protocol X"]` |
