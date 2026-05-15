# Discover: what can Bundie price for me?

You're integrating Bundie and need to know which event markets exist before you wire reads into your agent. `list_events` returns the live catalog with current YES prices, depth, and status — call it once at startup, cache the result, and pick the `event_id`s your agent cares about.

## Tool sequence

```
list_events                          # full catalog
list_events { status: "active" }     # filter to tradeable markets
list_events { resolver: "..." }      # filter by resolver class
get_event_detail { event_id }        # drill into one market's resolution rules
```

## Example call

```jsonc
// list_events { status: "active" }
{
  "count": 3,
  "events": [
    {
      "event_id": "usdc_depeg_99c_30min_30d",
      "description": "USDC trades below $0.99 for 30+ minutes within the next 30 days",
      "market_kind": 7,
      "resolver_class": "pyth_threshold_duration",
      "price": 0.042,
      "depth_usd": 18420.55,
      "window_end": "2026-06-13T00:00:00Z",
      "status": "active"
    },
    {
      "event_id": "kamino_main_tvl_drop_50m_24h_90d",
      "description": "Kamino Main Market TVL drops by $50M within any 24h window in the next 90 days",
      "market_kind": 8,
      "resolver_class": "onchain_tvl_rolling_window",
      "price": 0.118,
      "depth_usd": 6740.10,
      "window_end": "2026-08-12T00:00:00Z",
      "status": "active"
    },
    {
      "event_id": "aws_us_east_1_incident_30min_30d",
      "description": "AWS us-east-1 publishes an outage incident lasting 30+ minutes in the next 30 days",
      "market_kind": 9,
      "resolver_class": "aws_health_incident",
      "price": 0.087,
      "depth_usd": 2110.00,
      "window_end": "2026-06-13T00:00:00Z",
      "status": "active"
    }
  ]
}
```

## What each field means

- `event_id` — the stable slug. Pass this to `read_price`, `get_event_detail`, `resolver_track_record`.
- `market_kind` — numeric variant of the underlying LMSR market type. Informational; you rarely branch on it.
- `resolver_class` — which resolver settles this market. Tells you what the YES price is measuring (a Pyth feed, a TVL drop, an AWS incident). Filter by this to find all markets of a class.
- `price` — the current YES consensus probability in [0..1]. Stale by definition; refresh via `read_price`.
- `depth_usd` — total USD across both sides of the book. Low depth means the price is easy to move; weight it accordingly.
- `window_end` — when the market stops accepting trades and resolves. Reads against a market after its window has closed will still work but the price stops moving.

## Common pitfalls

- **Don't poll `list_events` on a hot loop.** It scans the whole catalog and isn't priced for high-frequency calls. Cache it at startup, refresh hourly at most.
- **Don't treat `price` from `list_events` as authoritative.** It's a snapshot. For any agent decision, use `read_price`, which is fresh and signed.
- **Resolved markets stay in the catalog.** Filter `status: "active"` if you only want tradeable markets — otherwise you'll wire reads against markets that won't move.
