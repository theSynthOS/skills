# SynthOS Agent Skills

Agent skills for managing DeFi yield through [SynthOS](https://synthos.fun) — AI-powered yield routing across EVM chains.

## Skills

### [synthos-yield-manager](skills/synthos-yield-manager/)
For users managing DeFi yield through conversation. Covers deposits, withdrawals, wallet analysis, AI recommendations, portfolio monitoring, and rebalancing.

### [synthos-yield-developer](skills/synthos-yield-developer/)
For developers integrating SynthOS yield routing into their own agents and applications. API reference, code examples, and integration patterns.

## Install

```bash
npx skills add theSynthOS/skills
```

Works with Claude Code, Cursor, OpenAI Codex, OpenCode, and any tool supporting the [Agent Skills](https://agentskills.io) specification.

## MCP Server

These skills work best with the SynthOS MCP server connected. No API keys needed:

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

Or install via [Smithery](https://smithery.ai/server/theSynthOS/yield).

## Links

- [Website](https://synthos.fun)
- [Documentation](https://docs.synthos.fun/docs/ai)
- [npm](https://www.npmjs.com/package/@synthos/mcp)
- [Smithery](https://smithery.ai/server/theSynthOS/yield)
