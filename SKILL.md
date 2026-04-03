---
name: bundie-yield
description: Manage DeFi yield through Bundie. Deposit into AI-optimized yield bundles, analyze wallets for risk profiling, get personalized recommendations via bull/bear/moderator debate, monitor positions, rebalance portfolios, buy crypto with fiat, and set rules — all through conversation. Works across EVM chains with 13 tools and 5 workflow prompts.
allowed-tools: bundie__bundie_check_yields bundie__bundie_portfolio bundie__bundie_deposit bundie__bundie_withdraw bundie__bundie_strategy_deposit bundie__bundie_strategy_withdraw bundie__bundie_get_recommendation bundie__bundie_analyze_wallet bundie__bundie_rebalance bundie__bundie_set_preferences bundie__bundie_migrate bundie__bundie_get_risk_scores bundie__bundie_buy_crypto
compatibility: Requires Bundie MCP server. Connect to hosted server (no API keys needed) or self-host via npm @bundie/mcp.
metadata:
  category: finance
  homepage: https://bundie.fi
  documentation: https://docs.bundie.fi/docs/ai/overview
  mcp-server: https://mcp.bundie.fi/mcp
  smithery: https://smithery.ai/server/Bundie/yield
---

# Bundie Yield

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
| Deposit or withdraw funds | [skills/bundie-yield-manager/references/deposit.md](skills/bundie-yield-manager/references/deposit.md) | "deposit 500 USDC", "withdraw my funds" |
| Analyze a wallet | [skills/bundie-yield-manager/references/analyze.md](skills/bundie-yield-manager/references/analyze.md) | "analyze my wallet", "what's my risk profile" |
| Get yield recommendations | [skills/bundie-yield-manager/references/optimize.md](skills/bundie-yield-manager/references/optimize.md) | "where should I put my USDC", "best yields for me" |
| Rebalance positions | [skills/bundie-yield-manager/references/optimize.md](skills/bundie-yield-manager/references/optimize.md) | "should I rebalance", "optimize my portfolio" |
| Check positions or performance | [skills/bundie-yield-manager/references/monitor.md](skills/bundie-yield-manager/references/monitor.md) | "show my portfolio", "how are my yields" |
| Find migration opportunities | [skills/bundie-yield-manager/references/optimize.md](skills/bundie-yield-manager/references/optimize.md) | "any better yields", "migration opportunities" |
| Set preferences or rules | [skills/bundie-yield-manager/references/optimize.md](skills/bundie-yield-manager/references/optimize.md) | "only audited protocols", "max 30% per protocol" |
| Browse available yields | [skills/bundie-yield-manager/references/monitor.md](skills/bundie-yield-manager/references/monitor.md) | "what yields are available", "highest APY" |
| Check risk scores | [skills/bundie-yield-manager/references/monitor.md](skills/bundie-yield-manager/references/monitor.md) | "show risk breakdown", "which protocols are safest" |
| Buy crypto with fiat | [skills/bundie-yield-manager/references/deposit.md](skills/bundie-yield-manager/references/deposit.md) | "I want to buy USDC", "invest $500" |
| API reference | [skills/bundie-yield-developer/references/api-reference.md](skills/bundie-yield-developer/references/api-reference.md) | "show me the API", "how to integrate" |
| Integration examples | [skills/bundie-yield-developer/references/examples.md](skills/bundie-yield-developer/references/examples.md) | "build a yield agent", "DAO treasury example" |

## Rules

1. Always confirm deposit/withdraw amounts with the user before executing
2. Show risk scores alongside APY — never recommend on APY alone
3. When user asks "where to earn yield", call `bundie_get_recommendation` (not just `bundie_check_yields`)
4. Amounts are human-readable (e.g., "100 USDC", not "100000000")
5. Default chain is Scroll (534352) unless user specifies otherwise
6. For first-time users, suggest `bundie_analyze_wallet` before recommendations
7. Cross-chain operations take 5-15 minutes to settle via LayerZero — always inform the user

## Quick Start Flow

For a new user asking about yield:

1. `bundie_analyze_wallet` — understand their risk profile
2. `bundie_get_recommendation` — AI-optimized bundle
3. Confirm with user
4. `bundie_deposit` + `bundie_strategy_deposit` — execute
