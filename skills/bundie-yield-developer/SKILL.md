---
name: bundie-yield-developer
description: Integrate Bundie yield routing into applications and agents. API reference for 12 MCP tools, code examples for yield agents, trading bots, DAO treasuries, and migration agents. Covers hosted and self-hosted setup.
allowed-tools: Bash(npm:*) Bash(npx:*) bundie__yields.check bundie__portfolio.view bundie__vault.deposit bundie__vault.withdraw bundie__strategy.deposit bundie__strategy.withdraw bundie__wallet.recommend bundie__wallet.analyze bundie__portfolio.rebalance bundie__portfolio.preferences bundie__wallet.migrate bundie__yields.risk_scores
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
| `yields.check` | Read | Browse yields with APY, risk, TVL |
| `portfolio.view` | Read | Positions, allocation %, weighted APY |
| `yields.risk_scores` | Read | Risk component breakdown per protocol |
| `vault.deposit` | Write | Deposit to Bundie vault |
| `vault.withdraw` | Write | Withdraw from vault |
| `strategy.deposit` | Write | Cross-chain strategy deposit |
| `strategy.withdraw` | Write | Withdraw from strategy |
| `wallet.analyze` | AI | Full wallet analysis + risk profile |
| `wallet.recommend` | AI | AI bundle via debate engine |
| `portfolio.rebalance` | AI | Optimal allocation comparison |
| `wallet.migrate` | AI | External DeFi migration opportunities |
| `portfolio.preferences` | State | Session yield rules |

See [references/api-reference.md](references/api-reference.md) for full input schemas.
See [references/examples.md](references/examples.md) for integration patterns.
