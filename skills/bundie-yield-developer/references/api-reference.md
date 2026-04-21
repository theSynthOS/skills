# Bundie MCP API Reference

## Authentication & Wallet Resolution

Bundie v2 supports two ways to identify the calling wallet:

**OAuth (recommended):** Sign in once at `https://auth.bundie.fi/authorize?redirect_uri=<your_callback>&state=<nonce>`. After sign-in, the MCP session carries your wallet automatically — no `walletAddress` needed per call.

**Explicit:** Pass `walletAddress` directly on each tool call. Useful for server-to-server integrations.

When both are present, the explicit `walletAddress` wins.

---

## Available Tools (16)

### Read-Only Tools

#### `yields_check`
Browse current DeFi yield opportunities across EVM chains.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `token` | string | No | Filter by token symbol (USDC, USDT) |
| `minApy` | number | No | Minimum APY (decimal, 0.05 = 5%) |
| `chain` | string | No | Filter by chain name (base, arbitrum) |
| `sortBy` | "apy" \| "name" | No | Sort field (default: apy) |
| `limit` | number | No | Max results (default: 10) |

#### `yields_risk_scores`
Detailed risk component breakdown per protocol — security, liquidity, maturity, centralization.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `protocolIds` | string[] | No | Filter to specific protocol UUIDs |
| `chain` | string | No | Filter by chain |
| `limit` | number | No | Max results (default: 20) |

#### `portfolio_view`
Check current positions, allocation breakdown, and weighted APY.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | No | Ethereum address — omit if signed in via OAuth |
| `chainId` | number | No | Chain ID (default: 534352) |

#### `wallet_balance`
Check token balance of a wallet on a specific chain.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | No | Ethereum address — omit if signed in via OAuth |
| `token` | string | No | Token symbol (default: USDC) |
| `chainId` | number | No | Chain ID (default: 534352 = Scroll) |

#### `bridge_status`
Check the status of a Relay Protocol bridge request.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `requestId` | string | Yes | Request ID returned by `bridge_to_scroll` |

### Write Tools

#### `vault_deposit`
Deposit assets into Bundie vault.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | No | Your wallet address — omit if signed in via OAuth |
| `asset` | string | Yes | Token symbol or address |
| `amount` | string | Yes | Human-readable amount (e.g., "100.5") |
| `chainId` | number | No | Chain ID (default: 534352) |

#### `vault_withdraw`
Withdraw assets from Bundie vault.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | No | Your wallet address — omit if signed in via OAuth |
| `asset` | string | Yes | Token symbol or address |
| `amount` | string | Yes | Human-readable amount |
| `recipientAddress` | string | No | Withdraw to different address |
| `chainId` | number | No | Chain ID (default: 534352) |

#### `strategy_deposit`
Deposit into a specific cross-chain yield strategy.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | No | Your wallet address — omit if signed in via OAuth |
| `protocolId` | string | Yes | Protocol UUID from yields.check |
| `amount` | string | Yes | Human-readable amount |
| `asset` | string | No | Token symbol (default: USDC) |
| `chainId` | number | No | Chain ID (default: 534352) |

#### `strategy_withdraw`
Withdraw from a specific strategy position.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | No | Your wallet address — omit if signed in via OAuth |
| `positionIndex` | number | Yes | Position index from portfolio |
| `amount` | string | Yes | Human-readable amount |
| `asset` | string | No | Token symbol (default: USDC) |
| `destinationChain` | number | Yes | Chain ID to receive funds |
| `chainId` | number | No | Source chain ID (default: 534352) |

#### `bridge_to_scroll`
Bridge USDC or USDT from any supported chain to Scroll via Relay Protocol.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | No | Your wallet address — omit if signed in via OAuth |
| `fromChain` | string | Yes | Source chain (base, arbitrum, optimism, ethereum, polygon) |
| `token` | string | No | Token to bridge (default: USDC) |
| `amount` | string | Yes | Human-readable amount (e.g., "100") |

#### `yields_buy`
Generate a payment link to buy USDC/USDT with a credit card via Ramp Network or Transak.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | No | Destination wallet — omit if signed in via OAuth |
| `token` | string | No | Token to buy (USDC or USDT, default: USDC) |
| `amount` | number | No | Fiat amount in USD |
| `provider` | "ramp" \| "transak" | No | Preferred provider |

### AI/Composed Tools

#### `wallet_analyze`
Full AI wallet analysis — risk profile, behavior, positions, idle assets.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | No | Wallet to analyze — omit if signed in via OAuth |

**Note:** Takes 1-3 minutes for new wallets. Results cached for 30 days.

#### `wallet_recommend`
AI-recommended diversified yield bundle via bull/bear/moderator debate.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | No | Wallet to recommend for — omit if signed in via OAuth |
| `tokens` | string[] | No | Filter by tokens |
| `chains` | number[] | No | Filter by chain IDs |
| `minRiskScore` | number | No | Min risk score (0-100) |
| `maxRiskScore` | number | No | Max risk score (0-100) |
| `bundleSize` | number | No | Strategies per bundle (1-10) |
| `excludeUSX` | boolean | No | Exclude USX token |

#### `portfolio_rebalance`
Compare current vs optimal allocation with optional auto-execute.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | No | Your wallet address — omit if signed in via OAuth |
| `chainId` | number | No | Chain ID (default: 534352) |
| `autoExecute` | boolean | No | Execute rebalance (default: false) |

#### `wallet_migrate`
Find migration opportunities from external DeFi to Bundie.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | No | Wallet to check — omit if signed in via OAuth |

### State Tool

#### `portfolio_preferences`
Set yield selection rules for the session.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | No | Wallet address — omit if signed in via OAuth |
| `maxAllocationPerProtocol` | number | No | Max % per protocol |
| `auditedOnly` | boolean | No | Only audited protocols |
| `minRiskScore` | number | No | Min risk score (0-100) |
| `maxRiskScore` | number | No | Max risk score (0-100) |
| `excludeChains` | number[] | No | Chain IDs to exclude |
| `excludeProtocols` | string[] | No | Protocol names to exclude |
| `preferredTokens` | string[] | No | Preferred tokens |

---

## x402 Payment Gate

The hosted MCP server charges **0.001 USDC per tool call** (paid in USDC on Base) for write and AI tools. Read-only tools are always free:

**Free:** `yields_check`, `yields_risk_scores`, `wallet_balance`, `bridge_status`

**Paid (0.001 USDC/call):** all other tools

Payments use the [x402 protocol](https://x402.org). Compatible MCP clients handle payment automatically via `X-PAYMENT` header.

---

## MCP Server Setup

### Hosted (recommended — no API keys needed)

Connect to the Bundie hosted MCP server:

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

Works with Claude Desktop, Cursor, VS Code, and any MCP-compatible client.

### Self-Hosted (for developers running their own instance)

```json
{
  "mcpServers": {
    "bundie": {
      "command": "npx",
      "args": ["-y", "@bundie/evm-mcp"],
      "env": {
        "BACKEND_URL": "https://backend.bundie.fi",
        "BACKEND_API_KEY": "your-key",
        "ANALYZER_URL": "https://ai.bundie.fi",
        "ANALYZER_API_KEY": "your-key"
      }
    }
  }
}
```

### Self-Hosted HTTP Mode

```bash
TRANSPORT=http PORT=3100 \
BACKEND_URL=https://backend.bundie.fi \
BACKEND_API_KEY=your-key \
ANALYZER_URL=https://ai.bundie.fi \
ANALYZER_API_KEY=your-key \
npx @bundie/evm-mcp
```

Then connect MCP clients to `http://localhost:3100/mcp`.
