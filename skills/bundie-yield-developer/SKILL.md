---
name: bundie-yield-developer
description: Integrate Bundie yield routing into applications and agents. API reference for 12 MCP tools, code examples for yield agents, trading bots, DAO treasuries, and migration agents. Covers hosted and self-hosted setup.
allowed-tools: Bash(npm:*) Bash(npx:*) bundie__check_yields bundie__portfolio bundie__deposit bundie__withdraw bundie__strategy_deposit bundie__strategy_withdraw bundie__get_recommendation bundie__analyze_wallet bundie__rebalance bundie__set_preferences bundie__migrate bundie__get_risk_scores
compatibility: Requires Bundie MCP server. Connect to hosted server (no API keys needed) or self-host via npm @bundie/mcp.
metadata:
  category: finance
  homepage: https://bundie.fi
  documentation: https://docs.bundie.fi/docs/ai/overview
  mcp-server: https://mcp.bundie.fi/mcp
---

# Bundie Developer Integration

Integrate Bundie yield routing into your agents and applications. Bundie exposes 12 MCP tools for yield discovery, risk analysis, AI recommendations, deposits, withdrawals, and portfolio management.

## Setup

### Hosted (no API keys needed)

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

### Self-Hosted

```bash
npm install @bundie/mcp
```

```json
{
  "mcpServers": {
    "bundie": {
      "command": "npx",
      "args": ["-y", "@bundie/mcp"],
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
| `check_yields` | Read | Browse yields with APY, risk, TVL |
| `portfolio` | Read | Positions, allocation %, weighted APY |
| `get_risk_scores` | Read | Risk component breakdown per protocol |
| `deposit` | Write | Deposit to Bundie vault |
| `withdraw` | Write | Withdraw from vault |
| `strategy_deposit` | Write | Cross-chain strategy deposit |
| `strategy_withdraw` | Write | Withdraw from strategy |
| `analyze_wallet` | AI | Full wallet analysis + risk profile |
| `get_recommendation` | AI | AI bundle via debate engine |
| `rebalance` | AI | Optimal allocation comparison |
| `migrate` | AI | External DeFi migration opportunities |
| `set_preferences` | State | Session yield rules |

See [references/api-reference.md](references/api-reference.md) for full input schemas.
See [references/examples.md](references/examples.md) for integration patterns.
