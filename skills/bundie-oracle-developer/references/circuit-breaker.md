# Circuit breaker: de-risk when Bundie crosses a threshold

You're running an agent that holds exposure to some protocol — Kamino liquidity, a USDC-denominated yield position, an AWS-hosted oracle relayer — and you want a forward-looking signal to pull out *before* the bad thing happens. This is the canonical Big Mario flow: Bundie's YES probability is the kill switch.

## Tool sequence

```
list_events                              # find the markets that match your exposure
get_event_detail { event_id }            # confirm what triggers a YES (one-time)
loop every 30s:
  read_price { event_id }                # fresh signed read
  verify_attestation { ... }             # trust gate
  if price > threshold: de-risk          # your agent's exit path
```

## Example: Kamino TVL drop circuit breaker

The agent holds 100k USDC in Kamino Main Market. It wants to withdraw if Bundie's consensus probability of a $50M TVL drop within 24h exceeds 15%.

```jsonc
// 1) get_event_detail { event_id: "kamino_main_tvl_drop_50m_24h_90d" }
{
  "event_id": "kamino_main_tvl_drop_50m_24h_90d",
  "description": "Kamino Main Market TVL drops by $50M within any 24h window in the next 90 days",
  "market_kind_proposed": "binary_yes_no",
  "resolver_class": "onchain_tvl_rolling_window",
  "outcome_yes": "Kamino Main Market reserves.total_supply_usd drops by >= $50M across any 24h window before window_end",
  "outcome_no": "No 24h window meets the threshold before window_end",
  "notes": "Resolver reads Kamino lending reserves account every 5 minutes.",
  "resolver_config": {
    "protocol": "kamino_main",
    "metric": "total_supply_usd",
    "drop_usd": 50000000,
    "window_seconds": 86400
  },
  "market_address": "9aZk...market_pubkey..."
}

// 2) read_price (polled every 30s)
{
  "event_id": "kamino_main_tvl_drop_50m_24h_90d",
  "price": 0.182,
  "confidence": 0.74,
  "depth_usd": 6740.10,
  "twap_24h": 0.094,
  "last_change_24h": 0.088,
  "spot_vs_twap_pct": 93.6,
  "resolver_track_record": { "total": 12, "disputed": 0, "lost": 0 },
  "signed_attestation": "9pQ2...base64...==",
  "as_of": "2026-05-14T11:42:08Z"
}
// price = 0.182 > 0.15 → agent withdraws
```

## Threshold design

You're choosing two things: the trigger value and the hysteresis.

- **Trigger value.** A higher threshold = fewer false alarms, more missed events. For a circuit breaker that costs you a few bps in slippage to trip, err low (5–10%). For a hard-to-reverse action (close a loan, sell the position), err high (20–40%).
- **Hysteresis.** Don't trip on a single spike. Require `price > threshold` for N consecutive reads, or require `spot_vs_twap_pct > X` so you're acting on a real move and not a single thin trade. Both filters survive the noise from low-depth markets.

```ts
// pseudo-code with hysteresis
let breaches = 0;
async function tick() {
  const p = await pollBundie("kamino_main_tvl_drop_50m_24h_90d");
  if (p.price > 0.15 && p.spot_vs_twap_pct > 30) breaches++;
  else breaches = 0;
  if (breaches >= 3) await agent.deriskKaminoPosition();
}
```

## Common pitfalls

- **Don't pick a market whose resolver doesn't match your risk.** A market measuring a 50M TVL drop won't fire on a 30M drain — read `resolver_config` and pick the market that maps to your actual exposure.
- **Don't trip without checking depth.** A 30% YES price on a $200-depth market can be one trader. Require `depth_usd > $1k` and `unique_traders_24h > 3` before trusting the move.
- **Don't ignore the resolver track record.** If `track_record.total = 0`, no settlement has happened on this resolver class yet. Use the price as advisory only; combine with other signals before pulling the trigger.
