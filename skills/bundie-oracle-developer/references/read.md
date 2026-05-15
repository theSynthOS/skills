# Read: live consensus price

`read_price` is the hero call. It returns the live YES probability for an event market, plus the depth, 24h TWAP, recent move, resolver track record, and an ed25519 signed attestation. This is what you wire into your agent's decision loop.

## Tool sequence

```
read_price { event_id }              # fresh, signed
verify_attestation { ... }           # confirm the read came from Bundie
                                     # (see references/verify-attestation.md)
```

## Example call

```jsonc
// read_price { event_id: "usdc_depeg_99c_30min_30d" }
{
  "event_id": "usdc_depeg_99c_30min_30d",
  "description": "USDC trades below $0.99 for 30+ minutes within the next 30 days",
  "window_start": "2026-05-14T00:00:00Z",
  "window_end": "2026-06-13T00:00:00Z",
  "price": 0.0421,
  "confidence": 0.81,
  "depth_usd": 18420.55,
  "trade_count_24h": 142,
  "unique_traders_24h": 37,
  "twap_24h": 0.039,
  "last_change_24h": 0.0031,
  "spot_vs_twap_pct": 7.95,
  "resolver_class": "pyth_threshold_duration",
  "resolver_track_record": { "total": 48, "disputed": 1, "lost": 0 },
  "signed_attestation": "3xK9...base64...==",
  "as_of": "2026-05-14T11:42:08Z"
}
```

## What each field means

- `price` — the YES consensus probability in [0..1]. Your primary signal.
- `confidence` — a scalar in [0..1] reflecting depth-weighted certainty. Low confidence with a non-zero price means the market is thin; weight your decision accordingly.
- `twap_24h` — 24-hour time-weighted average. Compare against `price` (`spot_vs_twap_pct`) to detect freshly-moving markets vs settled consensus.
- `last_change_24h` — absolute YES probability change over 24h. A market that moved from 0.01 to 0.04 in 24h is a different signal than one that has been at 0.04 for a week.
- `depth_usd` — total USD across both book sides. Below ~$1k, treat the price as advisory.
- `resolver_track_record` — `{ total, disputed, lost }`. See [resolver-trust.md](resolver-trust.md).
- `signed_attestation` — base64 or hex ed25519 signature over the response. Always verify before acting.
- `as_of` — backend timestamp at which the snapshot was sealed. Stale `as_of` (>30s old) indicates the backend is degraded.

## Wiring into an agent loop

```ts
// pseudo-code; your MCP client provides the actual call shape
async function pollBundie(eventId: string) {
  const price = await mcp.call("read_price", { event_id: eventId });

  const valid = await mcp.call("verify_attestation", {
    event_id: price.event_id,
    as_of: price.as_of,
    price: price.price,
    signed_attestation: price.signed_attestation,
    payload: price,
  });
  if (!valid.valid) throw new Error("attestation failed");

  return price;
}

setInterval(() => pollBundie("usdc_depeg_99c_30min_30d"), 30_000);
```

## Common pitfalls

- **Don't act on `price` without `verify_attestation`.** A spoofed MCP response or a tampered proxy can return any number it wants — the signature is your only proof Bundie produced it.
- **Don't poll faster than the market moves.** LMSR consensus updates on every trade; intra-trade polling burns x402 budget without new information. 15–60s is the realistic range for circuit-breaker agents.
- **Don't ignore `confidence` and `depth_usd`.** A `price` of 0.30 on a market with $200 of depth and 2 traders is noise, not signal.
