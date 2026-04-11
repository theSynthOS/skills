# Bundie MCP API Reference

## Available Tools

### Read-Only Tools

#### `yields.check`
Browse current DeFi yield opportunities across EVM chains.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `token` | string | No | Filter by token symbol (USDC, USDT) |
| `minApy` | number | No | Minimum APY (decimal, 0.05 = 5%) |
| `chain` | string | No | Filter by chain name (base, arbitrum) |
| `sortBy` | "apy" \| "name" | No | Sort field (default: apy) |
| `limit` | number | No | Max results (default: 10) |

#### `portfolio.view`
Check current positions, allocation breakdown, and weighted APY.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | Yes | Ethereum address (0x...) |
| `chainId` | number | No | Chain ID (default: 534352) |

### Write Tools

#### `vault.deposit`
Deposit assets into Bundie vault.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | Yes | Your wallet address |
| `asset` | string | Yes | Token symbol or address |
| `amount` | string | Yes | Human-readable amount (e.g., "100.5") |
| `chainId` | number | No | Chain ID (default: 534352) |

#### `vault.withdraw`
Withdraw assets from Bundie vault.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | Yes | Your wallet address |
| `asset` | string | Yes | Token symbol or address |
| `amount` | string | Yes | Human-readable amount |
| `recipientAddress` | string | No | Withdraw to different address |
| `chainId` | number | No | Chain ID (default: 534352) |

#### `strategy.deposit`
Deposit into a specific cross-chain yield strategy.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | Yes | Your wallet address |
| `protocolId` | string | Yes | Protocol UUID from yields.check |
| `amount` | string | Yes | Human-readable amount |
| `asset` | string | No | Token symbol (default: USDC) |
| `chainId` | number | No | Chain ID (default: 534352) |

#### `strategy.withdraw`
Withdraw from a specific strategy position.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | Yes | Your wallet address |
| `positionIndex` | number | Yes | Position index from portfolio |
| `amount` | string | Yes | Human-readable amount |
| `asset` | string | No | Token symbol (default: USDC) |
| `destinationChain` | number | Yes | Chain ID to receive funds |
| `chainId` | number | No | Source chain ID (default: 534352) |

### AI/Composed Tools

#### `wallet.analyze`
Full AI wallet analysis — risk profile, behavior, positions, idle assets.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | Yes | Wallet to analyze |

**Note:** Takes 1-3 minutes for new wallets. Results cached for 30 days.

#### `wallet.recommend`
AI-recommended diversified yield bundle via bull/bear/moderator debate.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | Yes | Wallet to recommend for |
| `tokens` | string[] | No | Filter by tokens |
| `chains` | number[] | No | Filter by chain IDs |
| `minRiskScore` | number | No | Min risk score (0-100) |
| `maxRiskScore` | number | No | Max risk score (0-100) |
| `bundleSize` | number | No | Strategies per bundle (1-10) |
| `excludeUSX` | boolean | No | Exclude USX token |

#### `portfolio.rebalance`
Compare current vs optimal allocation with optional auto-execute.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | Yes | Your wallet address |
| `chainId` | number | No | Chain ID (default: 534352) |
| `autoExecute` | boolean | No | Execute rebalance (default: false) |

#### `wallet.migrate`
Find migration opportunities from external DeFi to Bundie.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | Yes | Wallet to check |

### State Tool

#### `portfolio.preferences`
Set yield selection rules for the session.

**Inputs:**
| Param | Type | Required | Description |
|-------|------|----------|-------------|
| `walletAddress` | string | Yes | Wallet address |
| `maxAllocationPerProtocol` | number | No | Max % per protocol |
| `auditedOnly` | boolean | No | Only audited protocols |
| `minRiskScore` | number | No | Min risk score (0-100) |
| `maxRiskScore` | number | No | Max risk score (0-100) |
| `excludeChains` | number[] | No | Chain IDs to exclude |
| `excludeProtocols` | string[] | No | Protocol names to exclude |
| `preferredTokens` | string[] | No | Preferred tokens |

## Bridge Tools

### bridge.to_scroll
Bridge USDC/USDT from any EVM chain to Scroll.

**Input:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| walletAddress | string | yes | - | User wallet (0x...) |
| amount | string | yes | - | Amount to bridge e.g. "5000" |
| asset | "USDC" \| "USDT" | no | "USDC" | Token to bridge |
| sourceChain | string | no | "base" | Chain name or ID |

**Output:** `{ txHash, requestId, fees: { relayerFee, gasFee, totalFeeUsd }, estimatedTime, expectedOutput }`

**Notes:**
- Estimated time is ~5 seconds for Base/Arbitrum, ~2 minutes for Ethereum mainnet
- Tempo USDT is not supported -- use USDC only from Tempo

### bridge.status
Check bridge transfer status.

**Input:** `{ requestId: string }`
**Output:** `{ status: "pending" | "success" | "failure", txHashes? }`

**When to use:** After `bridge.to_scroll`, poll until `status = "success"` before proceeding to deposit.

## Wallet Tools (updated)

### wallet.balance (new)
Fast token balance check.

**Input:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| walletAddress | string | yes | - | User wallet (0x...) |
| asset | string | no | "USDC" | USDC, USDT, or ETH |
| chainId | number | no | 534352 | Chain ID (534352 = Scroll) |

**Output:** Human-readable balance string

**When to use:** Quick balance check. Much faster than `wallet.analyze`.

## MCP Server Setup

### Hosted (recommended — no API keys needed)

Connect to the Bundie hosted MCP server:

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

Works with Claude Desktop, Cursor, VS Code, and any MCP-compatible client.

### Self-Hosted (for developers running their own instance)

```json
{
  "mcpServers": {
    "bundie": {
      "command": "npx",
      "args": ["-y", "@bundie/mcp"],
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

### Self-Hosted HTTP Mode (deploy your own server)

```bash
TRANSPORT=http PORT=3100 \
BACKEND_URL=https://backend.bundie.fi \
BACKEND_API_KEY=your-key \
ANALYZER_URL=https://ai.bundie.fi \
ANALYZER_API_KEY=your-key \
npx @bundie/mcp
```

Then connect MCP clients to `http://localhost:3100/mcp`.

## Authentication

**Hosted server:** No authentication needed from users. API keys are managed server-side.

**Self-hosted:** Requires your own API keys:
- `BACKEND_API_KEY` — for vault/strategy operations
- `ANALYZER_API_KEY` — for wallet analysis and recommendations
- `MCP_AUTH_TOKEN` (optional, HTTP mode only) — protects the MCP endpoint
