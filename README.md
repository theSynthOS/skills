# Bundie Agent Skills

Agent skills for managing DeFi yield through [Bundie](https://bundie.fi) — AI-powered yield routing across EVM chains.

## Skills

### [bundie-yield-manager](skills/bundie-yield-manager/)
For users managing DeFi yield through conversation. Covers deposits, withdrawals, wallet analysis, AI recommendations, portfolio monitoring, and rebalancing.

### [bundie-yield-developer](skills/bundie-yield-developer/)
For developers integrating Bundie yield routing into their own agents and applications. API reference, code examples, and integration patterns.

## Install

```bash
npx skills add Bundie/skills
```

Works with Claude Code, Cursor, OpenAI Codex, OpenCode, and any tool supporting the [Agent Skills](https://agentskills.io) specification.

## MCP Server

These skills work best with the Bundie MCP server connected. No API keys needed:

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

Or install via [Smithery](https://smithery.ai/server/Bundie/yield).

## Links

- [Website](https://bundie.fi)
- [Documentation](https://docs.bundie.fi/docs/ai/overview)
- [npm](https://www.npmjs.com/package/@bundie/mcp)
- [Smithery](https://smithery.ai/server/Bundie/yield)
