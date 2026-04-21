---
name: bundie-yield-developer
description: >
  Integrate Bundie yield routing into applications and agents. API reference,
  code examples, and integration patterns for developers building on Bundie.
  Use when a developer asks about Bundie API, SDK, or integration.
allowed-tools: >
  Bash(npm:*),
  Bash(npx:*),
  bundie__yields_check,
  bundie__yields_risk_scores,
  bundie__yields_buy,
  bundie__portfolio_view,
  bundie__portfolio_preferences,
  bundie__portfolio_rebalance,
  bundie__vault_deposit,
  bundie__vault_withdraw,
  bundie__strategy_deposit,
  bundie__strategy_withdraw,
  bundie__wallet_analyze,
  bundie__wallet_recommend,
  bundie__wallet_migrate,
  bundie__wallet_balance,
  bundie__bridge_to_scroll,
  bundie__bridge_status
---

# Bundie Developer Integration

Bundie exposes DeFi yield routing as MCP tools that any AI agent can call. This skill helps developers integrate Bundie into their own agents and applications.

## Intent Routing

| Developer Intent | Reference | Example |
|-----------------|-----------|---------|
| Understand the API | [api-reference.md](references/api-reference.md) | "What tools does Bundie expose?" |
| See integration examples | [examples.md](references/examples.md) | "Show me how to build a yield agent" |
| Set up MCP connection | [api-reference.md](references/api-reference.md) | "How do I connect to Bundie MCP?" |
| Build a custom agent | [examples.md](references/examples.md) | "Build a trading bot that sweeps to yield" |

## Quick Start

### Connect via MCP (Claude Desktop / Cursor)

Add to your MCP config (no API keys needed):
```json
{
  "mcpServers": {
    "bundie": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.bundie.fi/evm"]
    }
  }
}
```

### Install as Agent Skill

```bash
npx skills add bundie-fi/mcp
```
