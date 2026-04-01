---
name: synthos-yield
description: Manage DeFi yield through SynthOS. Deposit into AI-optimized yield bundles, analyze wallets for risk profiling, get personalized recommendations via bull/bear/moderator debate, monitor positions, rebalance portfolios, buy crypto with fiat, and set rules — all through conversation. Works across EVM chains with 13 tools and 5 workflow prompts.
allowed-tools: synthos__synthos_check_yields synthos__synthos_portfolio synthos__synthos_deposit synthos__synthos_withdraw synthos__synthos_strategy_deposit synthos__synthos_strategy_withdraw synthos__synthos_get_recommendation synthos__synthos_analyze_wallet synthos__synthos_rebalance synthos__synthos_set_preferences synthos__synthos_migrate synthos__synthos_get_risk_scores synthos__synthos_buy_crypto
compatibility: Requires SynthOS MCP server. Connect to hosted server (no API keys needed) or self-host via npm @synthos/mcp.
metadata:
  category: finance
  homepage: https://synthos.fun
  documentation: https://docs.synthos.fun/docs/ai/overview
  mcp-server: https://mcp.synthos.fun/mcp
  smithery: https://smithery.ai/server/theSynthOS/yield
---

# SynthOS Yield

Manage DeFi yield through conversation. SynthOS indexes yield across EVM chains, risk-scores every opportunity via an AI engine, and constructs diversified bundles matched to your risk profile.

## Setup

Connect the SynthOS MCP server (no API keys needed):

```json
{
  "mcpServers": {
    "synthos": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.synthos.fun/mcp"]
    }
  }
}
```

## Intent Routing

| User Intent | Reference | Example Phrases |
|------------|-----------|-----------------|
| Deposit or withdraw funds | [skills/synthos-yield-manager/references/deposit.md](skills/synthos-yield-manager/references/deposit.md) | "deposit 500 USDC", "withdraw my funds" |
| Analyze a wallet | [skills/synthos-yield-manager/references/analyze.md](skills/synthos-yield-manager/references/analyze.md) | "analyze my wallet", "what's my risk profile" |
| Get yield recommendations | [skills/synthos-yield-manager/references/optimize.md](skills/synthos-yield-manager/references/optimize.md) | "where should I put my USDC", "best yields for me" |
| Rebalance positions | [skills/synthos-yield-manager/references/optimize.md](skills/synthos-yield-manager/references/optimize.md) | "should I rebalance", "optimize my portfolio" |
| Check positions or performance | [skills/synthos-yield-manager/references/monitor.md](skills/synthos-yield-manager/references/monitor.md) | "show my portfolio", "how are my yields" |
| Find migration opportunities | [skills/synthos-yield-manager/references/optimize.md](skills/synthos-yield-manager/references/optimize.md) | "any better yields", "migration opportunities" |
| Set preferences or rules | [skills/synthos-yield-manager/references/optimize.md](skills/synthos-yield-manager/references/optimize.md) | "only audited protocols", "max 30% per protocol" |
| Browse available yields | [skills/synthos-yield-manager/references/monitor.md](skills/synthos-yield-manager/references/monitor.md) | "what yields are available", "highest APY" |
| Check risk scores | [skills/synthos-yield-manager/references/monitor.md](skills/synthos-yield-manager/references/monitor.md) | "show risk breakdown", "which protocols are safest" |
| Buy crypto with fiat | [skills/synthos-yield-manager/references/deposit.md](skills/synthos-yield-manager/references/deposit.md) | "I want to buy USDC", "invest $500" |
| API reference | [skills/synthos-yield-developer/references/api-reference.md](skills/synthos-yield-developer/references/api-reference.md) | "show me the API", "how to integrate" |
| Integration examples | [skills/synthos-yield-developer/references/examples.md](skills/synthos-yield-developer/references/examples.md) | "build a yield agent", "DAO treasury example" |

## Rules

1. Always confirm deposit/withdraw amounts with the user before executing
2. Show risk scores alongside APY — never recommend on APY alone
3. When user asks "where to earn yield", call `synthos_get_recommendation` (not just `synthos_check_yields`)
4. Amounts are human-readable (e.g., "100 USDC", not "100000000")
5. Default chain is Scroll (534352) unless user specifies otherwise
6. For first-time users, suggest `synthos_analyze_wallet` before recommendations
7. Cross-chain operations take 5-15 minutes to settle via LayerZero — always inform the user

## Quick Start Flow

For a new user asking about yield:

1. `synthos_analyze_wallet` — understand their risk profile
2. `synthos_get_recommendation` — AI-optimized bundle
3. Confirm with user
4. `synthos_deposit` + `synthos_strategy_deposit` — execute
