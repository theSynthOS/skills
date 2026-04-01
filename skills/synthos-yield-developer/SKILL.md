---
name: synthos-yield-developer
description: Integrate SynthOS yield routing into applications and agents. API reference for 12 MCP tools, code examples for yield agents, trading bots, DAO treasuries, and migration agents. Covers hosted and self-hosted setup.
allowed-tools: Bash(npm:*) Bash(npx:*) synthos__synthos_check_yields synthos__synthos_portfolio synthos__synthos_deposit synthos__synthos_withdraw synthos__synthos_strategy_deposit synthos__synthos_strategy_withdraw synthos__synthos_get_recommendation synthos__synthos_analyze_wallet synthos__synthos_rebalance synthos__synthos_set_preferences synthos__synthos_migrate synthos__synthos_get_risk_scores
compatibility: Requires SynthOS MCP server. Connect to hosted server (no API keys needed) or self-host via npm @synthos/mcp.
metadata:
  category: finance
  homepage: https://synthos.fun
  documentation: https://docs.synthos.fun/docs/ai
  mcp-server: https://mcp.synthos.fun/mcp
---

# SynthOS Developer Integration

Integrate SynthOS yield routing into your agents and applications. SynthOS exposes 12 MCP tools for yield discovery, risk analysis, AI recommendations, deposits, withdrawals, and portfolio management.

## Setup

### Hosted (no API keys needed)

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

### Self-Hosted

```bash
npm install @synthos/mcp
```

```json
{
  "mcpServers": {
    "synthos": {
      "command": "npx",
      "args": ["-y", "@synthos/mcp"],
      "env": {
        "BACKEND_API_KEY": "your-key",
        "ANALYZER_API_KEY": "your-key"
      }
    }
  }
}
```

## Tools Overview

| Tool | Type | Description |
|------|------|-------------|
| `synthos_check_yields` | Read | Browse yields with APY, risk, TVL |
| `synthos_portfolio` | Read | Positions, allocation %, weighted APY |
| `synthos_get_risk_scores` | Read | Risk component breakdown per protocol |
| `synthos_deposit` | Write | Deposit to SynthOS vault |
| `synthos_withdraw` | Write | Withdraw from vault |
| `synthos_strategy_deposit` | Write | Cross-chain strategy deposit |
| `synthos_strategy_withdraw` | Write | Withdraw from strategy |
| `synthos_analyze_wallet` | AI | Full wallet analysis + risk profile |
| `synthos_get_recommendation` | AI | AI bundle via debate engine |
| `synthos_rebalance` | AI | Optimal allocation comparison |
| `synthos_migrate` | AI | External DeFi migration opportunities |
| `synthos_set_preferences` | State | Session yield rules |

See [references/api-reference.md](references/api-reference.md) for full input schemas.
See [references/examples.md](references/examples.md) for integration patterns.
