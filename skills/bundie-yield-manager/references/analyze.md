# Wallet Analysis & Risk Profiling

## Analyzing a Wallet

Use `bundie_analyze_wallet` to perform a comprehensive AI analysis of any EVM wallet.

```
bundie_analyze_wallet(walletAddress="0x...")
```

### What It Returns

- **Risk Profile:** One of 27 profiles based on on-chain behavior
- **Risk Level:** Conservative, Moderate, or Aggressive
- **Activity Summary:** Transaction count, protocols used, chains active on
- **DeFi Positions:** Current vault/lending positions with APY
- **Idle Assets:** Tokens sitting in wallet not earning yield
- **Executive Summary:** AI-generated overview of the wallet's DeFi activity

### Timing

- **First analysis:** Takes 1-3 minutes (scans 8 EVM chains)
- **Cached results:** Returns instantly (30-day cache)
- The tool handles polling automatically — just call it and wait

### When to Analyze

- Before getting recommendations (the AI needs the profile)
- When a user asks about their risk profile
- When a user wants to know what they're missing (idle assets)
- Before suggesting migrations from external DeFi

### After Analysis

Common next steps:
1. `bundie_get_recommendation` — uses the analysis to build a personalized yield bundle
2. `bundie_migrate` — find better yield vs current external positions
3. Share the risk profile and idle assets summary with the user

### Example Conversation

**User:** "Analyze my wallet 0xABC...DEF"

1. Call `bundie_analyze_wallet("0xABC...DEF")`
2. Present the risk profile, activity summary, and idle assets
3. Suggest: "You have $2,400 in idle USDC. Want me to find the best yield for your Moderate risk profile?"
4. If yes, call `bundie_get_recommendation`
