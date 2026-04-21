---
name: bundie-yield-manager
description: >
  Manage DeFi yield through Bundie — deposit into AI-optimized yield bundles,
  analyze wallets, get risk-scored recommendations, and monitor positions across
  EVM chains. Use when user asks about earning yield, DeFi returns, idle crypto,
  or managing investments.
allowed-tools: >
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

# Bundie Yield Manager

Bundie is a yield routing protocol that indexes yield across EVM chains, risk-scores every opportunity via an AI engine, and constructs diversified bundles matched to risk profiles.

## Intent Routing

| User Intent | Reference | Example Phrases |
|------------|-----------|-----------------|
| Deposit or withdraw funds | [deposit.md](references/deposit.md) | "deposit 500 USDC", "withdraw my funds", "put my USDC to work" |
| Analyze a wallet | [analyze.md](references/analyze.md) | "analyze my wallet", "what's my risk profile", "check 0x..." |
| Get yield recommendations | [optimize.md](references/optimize.md) | "where should I put my USDC", "best yields for me", "recommend strategies" |
| Rebalance positions | [optimize.md](references/optimize.md) | "should I rebalance", "optimize my portfolio", "any better allocations" |
| Check positions or performance | [monitor.md](references/monitor.md) | "show my portfolio", "how are my yields doing", "what's my APY" |
| Find migration opportunities | [optimize.md](references/optimize.md) | "any better yields outside", "migration opportunities", "compare yields" |
| Set preferences or rules | [optimize.md](references/optimize.md) | "only audited protocols", "max 30% per protocol", "no risky vaults" |
| Browse available yields | [monitor.md](references/monitor.md) | "what yields are available", "show me yield options", "highest APY" |
| Bridge funds to Scroll | [deposit.md](references/deposit.md) | "bridge from Base", "move USDC to Scroll", "bring funds from Arbitrum" |
| Check wallet balance | [monitor.md](references/monitor.md) | "how much USDC do I have", "what's my balance on Scroll" |
| Buy crypto with card | [deposit.md](references/deposit.md) | "buy USDC", "purchase with credit card", "onramp" |

## Key Rules

1. **Always confirm** deposit/withdraw amounts with the user before executing
2. **Show risk scores alongside APY** — never recommend on APY alone
3. When user asks "where to earn yield", call `wallet_recommend` (not just `yields_check`)
4. **Amounts are human-readable** (e.g., "100 USDC", not "100000000")
5. Default chain is Scroll (534352) unless user specifies otherwise
6. For first-time users, suggest `wallet_analyze` before recommendations
7. Cross-chain operations take 5-15 minutes to settle via LayerZero — always inform the user

## Quick Start Flow

For a new user asking about yield:
1. `wallet_analyze` — understand their risk profile
2. `wallet_recommend` — AI-optimized bundle
3. Confirm with user, then `vault_deposit` + `strategy_deposit`
