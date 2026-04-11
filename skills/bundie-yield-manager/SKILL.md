---
name: bundie-yield-manager
description: Manage DeFi yield through Bundie. Deposit into AI-optimized yield bundles, analyze wallets for risk profiling, get personalized recommendations via bull/bear/moderator debate, monitor positions, rebalance portfolios, and set rules — all through conversation. Works across EVM chains. Requires Bundie MCP server connection.
allowed-tools: bundie__yields.check bundie__portfolio.view bundie__vault.deposit bundie__vault.withdraw bundie__strategy.deposit bundie__strategy.withdraw bundie__wallet.recommend bundie__wallet.analyze bundie__portfolio.rebalance bundie__portfolio.preferences bundie__wallet.migrate bundie__yields.risk_scores bundie__bridge.to_scroll bundie__bridge.status bundie__wallet.balance
compatibility: Requires Bundie MCP server. Connect to hosted server (no API keys needed) or self-host via npm @bundie/mcp.
metadata:
  category: finance
  homepage: https://bundie.fi
  documentation: https://docs.bundie.fi/docs/ai/overview
  mcp-server: https://mcp.bundie.fi/mcp
---

# Bundie Yield Manager

Manage DeFi yield through conversation. Bundie indexes yield across EVM chains, risk-scores every opportunity via an AI engine, and constructs diversified bundles matched to your risk profile.

## Setup

Connect the Bundie MCP server (no API keys needed):

```json
{
  "mcpServers": {
    "bundie": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.bundie.fi/mcp"]
    }
  }
}
```

## Intent Routing

| User Intent | Reference | Example Phrases |
|------------|-----------|-----------------|
| Deposit or withdraw funds | [references/deposit.md](references/deposit.md) | "deposit 500 USDC", "withdraw my funds" |
| Analyze a wallet | [references/analyze.md](references/analyze.md) | "analyze my wallet", "what's my risk profile" |
| Get yield recommendations | [references/optimize.md](references/optimize.md) | "where should I put my USDC", "best yields for me" |
| Rebalance positions | [references/optimize.md](references/optimize.md) | "should I rebalance", "optimize my portfolio" |
| Check positions or performance | [references/monitor.md](references/monitor.md) | "show my portfolio", "how are my yields" |
| Find migration opportunities | [references/optimize.md](references/optimize.md) | "any better yields", "migration opportunities" |
| Set preferences or rules | [references/optimize.md](references/optimize.md) | "only audited protocols", "max 30% per protocol" |
| Browse available yields | [references/monitor.md](references/monitor.md) | "what yields are available", "highest APY" |
| Check risk scores | [references/monitor.md](references/monitor.md) | "show risk breakdown", "which protocols are safest" |
| User has funds on another chain | [references/deposit.md](references/deposit.md) | bridge.to_scroll → bridge.status → deposit flow |
| "bridge from Base/Arbitrum/Tempo" | [references/deposit.md](references/deposit.md) | bridge.to_scroll |
| "what's my balance" | [references/monitor.md](references/monitor.md) | wallet.balance (faster than wallet.analyze) |
| "is my bridge done" / "bridge status" | [references/monitor.md](references/monitor.md) | bridge.status |

## Rules

1. Always confirm deposit/withdraw amounts with the user before executing
2. Show risk scores alongside APY — never recommend on APY alone
3. When user asks "where to earn yield", call `wallet.recommend` (not just `yields.check`)
4. Amounts are human-readable (e.g., "100 USDC", not "100000000")
5. Default chain is Scroll (534352) unless user specifies otherwise
6. For first-time users, suggest `wallet.analyze` before recommendations
7. Cross-chain operations take 5-15 minutes to settle via LayerZero — always inform the user

## Quick Start Flow

For a new user asking about yield:

1. `wallet.analyze` — understand their risk profile
2. `wallet.recommend` — AI-optimized bundle
3. Confirm with user
4. `vault.deposit` + `strategy.deposit` — execute

For a user bridging from another chain first, use the `bridge-and-earn` prompt to trigger the full bridge → deposit flow in one step.
