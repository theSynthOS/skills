# Resolver trust: should I act on this price?

Every Bundie market has a resolver — the on-chain or off-chain process that decides whether YES or NO settled. The price is meaningful only to the extent the resolver is reliable. `resolver_track_record` gives you `{ total, disputed, lost }` so your agent can grade trust before betting decisions on the read.

## Tool sequence

```
read_price { event_id }                  # track record returned inline
resolver_track_record { event_id }       # same data, no fresh price read
                                         # — use when you only want the
                                         # trust signal
```

## Example call

```jsonc
// resolver_track_record { event_id: "usdc_depeg_99c_30min_30d" }
{
  "event_id": "usdc_depeg_99c_30min_30d",
  "resolver_class": "pyth_threshold_duration",
  "track_record": {
    "total": 48,
    "disputed": 1,
    "lost": 0
  },
  "as_of": "2026-05-14T11:42:08Z"
}
```

## How to read the three numbers

- **`total`** — settlements this resolver has executed (across all markets of its class). The denominator for trust.
- **`disputed`** — settlements that were challenged via the protocol's dispute window. A non-zero count isn't fatal — disputes can resolve in the resolver's favour — but it's a flag.
- **`lost`** — settlements where the dispute concluded against the resolver and the outcome was overturned. A non-zero `lost` is the strongest negative signal.

## Suggested trust gates

| Track record | Action |
|---|---|
| `total = 0` | Treat the price as advisory. Don't gate critical decisions on this market alone. Combine with off-chain signals. |
| `total < 10, lost = 0` | Useful for low-stakes decisions. Pair with hysteresis and depth checks before acting. |
| `total >= 10, disputed = 0, lost = 0` | Acceptable for circuit-breaker triggers up to medium-stakes positions. |
| `total >= 50, disputed <= 2, lost = 0` | Acceptable for production hot loops including position de-risking. |
| `lost > 0` | Pause. Read [get_event_detail] to understand the resolver class, check whether the lost settlement was a bug since fixed, and decide deliberately. |

## Worked example

Your agent runs a $50k Kamino position and watches `kamino_main_tvl_drop_50m_24h_90d`. The price spikes to 0.22. Before tripping the circuit breaker:

```jsonc
// resolver_track_record returns:
{
  "track_record": { "total": 4, "disputed": 0, "lost": 0 }
}
```

`total = 4` means the resolver class is young. The right move is to:

1. Use the price as one input among several, not the trigger.
2. Lower the per-tick position you're willing to act on (e.g., partial withdrawal instead of full exit).
3. Read `get_event_detail` and confirm the `resolver_config` matches the risk you actually care about (correct protocol, correct metric, correct threshold).

## Common pitfalls

- **Don't conflate market depth with resolver trust.** A market with $50k depth and a brand-new resolver is still risky to settlement, regardless of how thick the book is.
- **Don't read the track record once and cache it forever.** Track record updates after every settlement; refresh on a multi-hour cadence at minimum, or every time you onboard a new market into your agent's watchlist.
- **Don't assume `lost = 0` means the resolver is correct.** It means it has never been overturned via dispute. A resolver class with no dispute participants is functionally untested. Lean on multiple signals until the protocol has visible disputers.
