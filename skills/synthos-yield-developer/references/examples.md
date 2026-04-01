# SynthOS Integration Examples

## Example 1: Simple Yield Agent

A basic agent that deposits idle USDC into the best yield:

```
1. synthos_analyze_wallet("0xUSER")
   → Risk profile: Moderate, $5,000 idle USDC detected

2. synthos_get_recommendation("0xUSER", tokens=["USDC"])
   → Bundle: 40% Morpho (Base, 9.1%), 35% Fluid (Scroll, 8.5%), 25% Moonwell (Optimism, 7.8%)

3. synthos_deposit("0xUSER", asset="USDC", amount="5000")
   → Funds in SynthOS vault

4. synthos_strategy_deposit("0xUSER", protocolId="morpho-uuid", amount="2000")
   synthos_strategy_deposit("0xUSER", protocolId="fluid-uuid", amount="1750")
   synthos_strategy_deposit("0xUSER", protocolId="moonwell-uuid", amount="1250")
   → Funds deployed across 3 protocols on 3 chains
```

## Example 2: Trading Bot + Yield Sweep

Combine with a trading agent — sweep profits into yield after trades:

```
After a profitable trade:

1. Check balance: synthos_portfolio("0xBOT")
2. If idle USDC > 1000:
   synthos_get_recommendation("0xBOT", tokens=["USDC"], minRiskScore=80)
3. Deposit excess: synthos_deposit("0xBOT", "USDC", excessAmount)
4. Allocate per recommendation
```

## Example 3: DAO Treasury Management

A DAO treasury agent with governance-set rules:

```
1. synthos_set_preferences("0xDAO",
     maxAllocationPerProtocol=20,
     auditedOnly=true,
     minRiskScore=85,
     preferredTokens=["USDC", "USDT"])

2. synthos_get_recommendation("0xDAO")
   → Conservative bundle respecting DAO rules

3. Deploy funds per recommendation

4. Weekly: synthos_rebalance("0xDAO")
   → Check if allocation is still optimal
```

## Example 4: Migration Agent

Help users move from external DeFi to SynthOS:

```
1. synthos_analyze_wallet("0xUSER")
   → Found 3 external positions with suboptimal yield

2. synthos_migrate("0xUSER")
   → Migration opportunities:
     Aave USDC (3.2%) → Morpho USDC (9.1%) = +5.9% improvement
     Compound USDT (2.8%) → Fluid USDT (7.5%) = +4.7% improvement

3. Present to user, confirm, then execute migrations
```

## Example 5: Risk-Managed Portfolio

An agent with stop-loss and risk monitoring:

```
1. synthos_set_preferences("0xUSER", minRiskScore=80, maxAllocationPerProtocol=25)
2. synthos_get_recommendation("0xUSER") → Deploy funds
3. Periodically:
   - synthos_portfolio("0xUSER") → Check positions
   - synthos_rebalance("0xUSER") → Compare vs optimal
   - If significant delta, notify user or auto-rebalance
```

## Building on SynthOS: Architecture Pattern

```
Your Agent / Application
    │
    ├─ Install: npx skills add theSynthOS/mcp
    │  or: Add MCP server to config
    │
    ├─ Read:  synthos_check_yields, synthos_portfolio
    ├─ Write: synthos_deposit, synthos_withdraw, synthos_strategy_*
    ├─ AI:    synthos_analyze_wallet, synthos_get_recommendation
    └─ State: synthos_set_preferences
         │
         ▼
    SynthOS MCP Server (@synthos/mcp)
         │
         ├─ Backend API (vault ops, strategies, protocols)
         └─ AI Analyzer (risk scoring, debate engine, recommendations)
              │
              ▼
         DeFi Protocols (Morpho, Fluid, Moonwell, Yearn, etc.)
         across EVM chains (Scroll, Base, Arbitrum, Optimism, Mode, ...)
```

## Getting API Keys

Contact the SynthOS team at [synthos.fun](https://synthos.fun) for API access, or check the docs at [docs.synthos.fun/mcp](https://docs.synthos.fun/docs/ai/mcp-server).
