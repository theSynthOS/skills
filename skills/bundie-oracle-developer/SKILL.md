---
name: bundie-oracle-developer
description: >
  Integrate Bundie as a forward-signal oracle into agents and applications.
  Bundie prices what's about to happen — depegs, outages, TVL drops — and
  exposes the live YES consensus probability as a signed read over MCP.
  Use when a developer asks about reading Bundie prices, wiring a circuit
  breaker, verifying attestations, or judging resolver trust.
allowed-tools: >
  Bash(npm:*),
  Bash(npx:*),
  bundie__list_events,
  bundie__read_price,
  bundie__get_event_detail,
  bundie__verify_attestation,
  bundie__resolver_track_record
compatibility: Requires Bundie Solana MCP server. Connect to hosted server (no API keys during devnet beta) or self-host via npm @bundie/sol-mcp.
metadata:
  category: oracle
  homepage: https://bundie.fi
  documentation: https://docs.bundie.fi/docs/oracle/overview
  mcp-server: https://mcp.bundie.fi/solana
  npm: https://www.npmjs.com/package/@bundie/sol-mcp
---

# Bundie Oracle Developer

Bundie is an oracle that agents read to price the future. Existing oracles price the present — BTC right now, ETH right now, USDC right now. Bundie prices what's *about* to happen: stablecoin depegs, cloud-provider outages, protocol TVL drops, oracle deviations. Every measurable event has a market; the YES price is the live consensus probability; an agent reads it for ~$0.001 a call over x402.

This skill teaches you how to wire that read into your own agent.

## Intent Routing

| Developer Intent | Reference | Example |
|-----------------|-----------|---------|
| Find what Bundie can price | [references/discover.md](references/discover.md) | "What event markets exist?", "Can Bundie price a USDC depeg?" |
| Read a live consensus price | [references/read.md](references/read.md) | "How do I read the YES probability?", "Wire read_price into my loop" |
| Build a de-risk trigger | [references/circuit-breaker.md](references/circuit-breaker.md) | "Pull liquidity when depeg risk > 5%", "Big Mario circuit breaker" |
| Verify a price is from Bundie | [references/verify-attestation.md](references/verify-attestation.md) | "How do I trust the read?", "Check the signature" |
| Judge whether to act on a price | [references/resolver-trust.md](references/resolver-trust.md) | "Is this resolver reliable?", "Should I bet on this market" |

## Setup

### Hosted (recommended — no API keys during devnet beta)

```json
{
  "mcpServers": {
    "bundie-sol": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.bundie.fi/solana"]
    }
  }
}
```

### Local stdio (Node 18+)

```json
{
  "mcpServers": {
    "bundie-sol": {
      "command": "npx",
      "args": ["-y", "@bundie/sol-mcp"]
    }
  }
}
```

### Self-hosted backend

```bash
BACKEND_URL=http://localhost:3001 \
BACKEND_AUTH=your-bearer-token \
npx @bundie/sol-mcp
```

## How an agent uses Bundie

```
agent startup
  ├─ list_events                            ← what can Bundie price for me?
  └─ get_event_detail (per candidate)       ← what triggers a YES?
agent steady state (every ~30s)
  ├─ read_price (for the markets you gated on)
  └─ verify_attestation                     ← did this come from Bundie?
agent decision
  └─ de-risk / open position / circuit-break based on the YES probability
```

You pay roughly `$0.001 × calls/month` for the read. Polling 5 markets every 30s for 30 days is ~$13. A single avoided liquidation pays back the year.

## Rules

1. **Always verify before acting on a price.** The `read_price` response includes a `signed_attestation`. If you skip `verify_attestation` you are trusting the transport, not Bundie.
2. **Read the resolver track record before betting decisions on a market.** A market with `{ total: 0 }` settlements has no history. See [references/resolver-trust.md](references/resolver-trust.md).
3. **Use `get_event_detail` once, cache it.** Resolver config (Pyth feed, threshold, window) doesn't change between settlements — there's no point re-fetching it inside your hot loop.
4. **Poll `read_price`, not `list_events`.** `list_events` is for discovery at startup. The hot path is `read_price` on the specific markets you care about.
5. **Treat YES as a probability, not a verdict.** A `price` of 0.04 means the market consensus is 4% — informative, not deterministic. Choose your action threshold.
6. **Cache the attestation public key.** It doesn't rotate within a deploy. `verify_attestation` caches it for you automatically; if you build your own verifier, do the same.
