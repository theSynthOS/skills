# Bundie Solana — Agent Skills

The Bundie Solana CLI (`bundie-sol`, npm: `@bundie/sol-cli`) is the agent-facing surface for composing strategies, opening prediction markets, and resolving outcomes on Solana devnet.

Humans back strategies and take prediction positions through the webapp (wrapped as a Seeker TWA). Agents do the rest — composition, market-making, resolution — through this CLI.

**Two usage modes, same CLI:**

- **Mode 1 — Human-triggered.** A human prompts Claude Desktop, Cursor, or any MCP-capable assistant with the skills file available. The agent reads this file, invokes `bundie-sol` through its shell tool, and returns results. The human is in the loop for intent and approval.
- **Mode 2 — Autonomous.** A cron job, ZerePy worker, elizaOS action, long-running headless `claude -p` loop, or custom Rust daemon operates on its own triggers (new strategy crossing TVL threshold, time elapsed, volatility below X) and invokes `bundie-sol` without a per-action human prompt.

The demo leads with Mode 2 (two autonomous agents in split-screen terminals) because it lands the "agents do the hard work" positioning hardest. First-time users typically start in Mode 1 via Claude Desktop.

---

## Archetypes

Three agent roles exist on the protocol. Any wallet can play any combination, subject to the creator self-exclusion rule (a strategy's creator cannot take positions on markets opened against their own strategy).

| Archetype | Role | Commands |
|---|---|---|
| **Strategy-creator** | Composes DeFi strategies through Beethoven (Kamino USDC lending for MVP). Earns performance fees from backer profits. | `create-strategy` |
| **Market-maker** | Opens LS-LMSR prediction markets on strategies *other* agents have created. Seeds initial liquidity. Different wallet from the strategy-creator by design — otherwise self-exclusion has no teeth. | `create-market` |
| **Resolver** | Permissionless operational role. Calls `resolve-market` after the resolution slot; the program reads the strategy's on-chain NAV oracle and settles. Typically a cron. | `resolve-market` |

Humans on the webapp handle the two consumption roles: **Backer** (`buy-shares`) and **Predictor** (`predict`). The CLI supports those commands too for autonomous backer / predictor agents, but they are not the demo hero.

---

## Prerequisites

### 1. Install

```bash
npm install -g @bundie/sol-cli
# or run ad-hoc
npx @bundie/sol-cli --help
```

Binary: `bundie-sol`.

### 2. Wallet keypair

```bash
solana-keygen new --outfile ~/.config/solana/id.json
```

Default path is `~/.config/solana/id.json`; override with `--keypair <path>`.

### 3. Devnet SOL (for tx fees)

```bash
solana airdrop 2 --url devnet
```

### 4. Devnet USDC (for deposits and predictions)

Devnet USDC mint: `4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU`

1. Create the ATA if you don't have one:
   ```bash
   spl-token create-account 4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU --url devnet
   ```
2. Faucet USDC at https://faucet.circle.com (select "Devnet").

Minimum: 1 USDC per strategy deposit; any positive amount for predictions.

### Global CLI flags

| Flag | Description | Default |
|------|-------------|---------|
| `--keypair <path>` | Solana keypair JSON path | `~/.config/solana/id.json` |
| `--rpc <url>` | Solana RPC endpoint | devnet public RPC |

---

# Archetype flows

## Strategy-creator

Composes DeFi strategies. In MVP, strategies deposit into Kamino USDC lending through Beethoven. Post-hackathon adds more Beethoven protocols (Jupiter Earn, Marginfi, Drift) and a perp action trait for delta-neutral funding capture.

### Mode 1 — Human-triggered in Claude Desktop

**User prompt:**
> "Compose a conservative USDC lending strategy on Kamino with a 10% performance fee. Seed it with 100 USDC."

**Expected agent behavior:**
1. Agent reads this skills file.
2. Agent runs:
   ```bash
   bundie-sol create-strategy \
     --name "Conservative Stable Yield" \
     --protocol kamino \
     --fee-bps 1000 \
     --deposit 100 \
     --min-deposit 1
   ```
3. Agent parses stdout for the `strategy:` address and returns it to the user.

**User prompt with less context:**
> "Launch a yield strategy for me."

**Expected agent behavior:** ask one clarifying question (protocol? fee? name?) or pick sensible defaults (Kamino, 10% fee, agent-generated name) and confirm before calling `create-strategy`.

### Mode 2 — Autonomous loop

A scheduled worker that composes a new strategy whenever observed devnet Kamino supply APY drops below a threshold:

```bash
#!/usr/bin/env bash
# cron: every 6 hours
APY=$(curl -s https://api.kamino.fi/devnet/reserves/usdc | jq -r .supply_apy)
if (( $(echo "$APY < 5.0" | bc -l) )); then
  bundie-sol create-strategy \
    --name "Kamino Recovery $(date +%s)" \
    --protocol kamino \
    --fee-bps 1000 \
    --deposit 50
fi
```

Or as a long-running `claude -p` loop that reads this file, observes its own triggers, and acts:

```
claude -p --model sonnet \
  --append-system "You are a strategy-creator agent. Read SKILLS.md. Every 6 hours, check Kamino APY via the RPC and compose a new strategy if conditions warrant. Only call bundie-sol create-strategy."
```

### Hard constraint

The wallet you create with becomes the strategy's `authority`. That wallet is then **blocked from taking positions on any prediction market opened against this strategy** (on-chain address-equality check). This is why the market-maker archetype is a separate wallet by design.

---

## Market-maker

Opens LS-LMSR prediction markets on strategies that other agents have created. Seeds initial liquidity so the market has depth from trade one.

**Must be a different wallet from the strategy's creator.** The `create-market` instruction itself does not enforce this, but subsequent `predict` / `buy_shares` calls on the market will revert with `CreatorCannotPredictOnOwnStrategy` if the market-maker is the strategy creator.

### Mode 1 — Human-triggered

**User prompt:**
> "Open a market on whether strategy `93amC4...` exceeds 10% APY by June 1."

**Expected agent behavior:**
1. Compute `resolution-days` from the target date.
2. Run:
   ```bash
   bundie-sol create-market \
     --strategy 93amC41qp6YfTQwsugjgRG21eUtmbqiCx4VJjzE2xZYF \
     --question "Will Kamino Seed Strategy exceed 10% APY by June 1?" \
     --threshold-bps 1000 \
     --resolution-days 40 \
     --initial-subsidy 10 \
     --fee-bps 100
   ```
3. Return the new `market:` address to the user.

### Mode 2 — Autonomous watch-loop

Poll `list-strategies` on an interval and open a market whenever a new strategy crosses $500 in TVL:

```bash
#!/usr/bin/env bash
# cron: every 1 hour
SEEN_FILE="/var/cache/bundie-markets-seen.txt"
touch "$SEEN_FILE"

bundie-sol list-strategies | awk '$3 >= 500 { print $1 }' | while read STRAT; do
  if ! grep -q "$STRAT" "$SEEN_FILE"; then
    bundie-sol create-market \
      --strategy "$STRAT" \
      --question "Will this strategy exceed 10% APY in 30 days?" \
      --threshold-bps 1000 \
      --resolution-days 30 \
      --initial-subsidy 10
    echo "$STRAT" >> "$SEEN_FILE"
  fi
done
```

### Choosing parameters

- **`--threshold-bps`** — The APY target. 1000 = 10%. Pick a level where the strategy's actual performance is plausibly either side.
- **`--resolution-days`** — How long backers and predictors have to trade before settlement. 7–30 days is typical.
- **`--initial-subsidy`** — USDC put into the LS-LMSR curve at creation. More subsidy = tighter spreads early, but locks capital. 10–50 USDC is typical.
- **`--fee-bps`** — Trading fee. 100 bps = 1%. Competitive range 50–200.

---

## Resolver

Permissionless. Anyone can call `resolve-market` once `market.resolution_slot` has been reached. The program reads the strategy's NavOracle PDA (seeds `["nav", strategy]`) and compares the TWAP value against the market's threshold. No external oracle in the resolution path.

### Mode 2 — Keeper cron

```bash
#!/usr/bin/env bash
# cron: every 10 minutes
# Resolve every market past its resolution slot. Markets already resolved
# will simply error out and we move on.

bundie-sol list-markets 2>/dev/null | awk '$STATUS == "active" && $RESOLUTION_SLOT <= $CURRENT_SLOT { print $MARKET }' | while read M; do
  bundie-sol resolve-market --market "$M" || true
done
```

`list-markets` is not yet a CLI command (post-hackathon); for now, keepers track markets via indexing events or a seeded address list.

### Mode 1 — Manual resolution

**User prompt:**
> "Resolve market `2XfBNn...` if it's past its resolution slot."

**Expected agent behavior:**
```bash
bundie-sol resolve-market --market 2XfBNnZ5YEH23tB4QdCNzXu2bJmeY5WwmMuikGgZP8wu
```

Agent returns the decoded outcome (YES / NO) from stdout.

---

# Participation commands (for autonomous backer / predictor agents)

These are reference — the demo and pitch show humans doing these via the webapp. Autonomous agents that want to express views on-chain still use the CLI.

## `buy-shares` (Backer)

Invest USDC into an existing strategy.

```bash
bundie-sol buy-shares \
  --strategy <pubkey> \
  --amount <usdc>
```

| Option | Description |
|---|---|
| `--strategy` | Strategy account address (required) |
| `--amount` | USDC amount to invest (required) |

Output includes the tx signature.

## `predict` (Predictor)

Buy YES or NO shares on an LS-LMSR market.

```bash
bundie-sol predict \
  --market <pubkey> \
  --side <yes|no> \
  --amount <usdc>
```

| Option | Description |
|---|---|
| `--market` | Market address (required) |
| `--side` | `yes` or `no` (required) |
| `--amount` | USDC to stake (required) |

**Blocked** (by `CreatorCannotPredictOnOwnStrategy`) if the calling wallet is the strategy's on-chain `authority`. This is the v5 spec's first-party address-equality check.

---

# Full command reference

## `create-strategy`

```bash
bundie-sol create-strategy \
  --name <name> \
  [--protocol <kamino|marginfi|jupiter|<pubkey>>] \
  [--fee-bps <bps>] \
  [--deposit <usdc>] \
  [--min-deposit <usdc>] \
  [--usdc-mint <pubkey>]
```

| Option | Default | Description |
|---|---|---|
| `--name` | — | Strategy name, max 32 chars (required) |
| `--protocol` | `kamino` | Protocol name or program address |
| `--fee-bps` | `1000` | Performance fee in basis points (1000 = 10%) |
| `--deposit` | `0` | Initial USDC deposit; `0` skips the deposit tx |
| `--min-deposit` | `1` | Minimum USDC any backer must deposit |
| `--usdc-mint` | devnet USDC | Collateral mint |

Sample output:
```
Creating strategy "Alpha Momentum" on devnet...
  wallet: <your-pubkey>

  create_strategy: <tx-sig>
  strategy: <strategy-address>
  mint:     <share-mint-address>

✓ Strategy created successfully.
```

## `create-market`

```bash
bundie-sol create-market \
  --strategy <pubkey> \
  --question <text> \
  --threshold-bps <bps> \
  [--resolution-days <days>] \
  [--initial-subsidy <usdc>] \
  [--fee-bps <bps>] \
  [--strategy-b <pubkey>]
```

| Option | Default | Description |
|---|---|---|
| `--strategy` | — | Strategy to open a market on (required) |
| `--question` | — | Market question (max 128 chars) |
| `--threshold-bps` | — | APY threshold in basis points |
| `--resolution-days` | `7` | Days until resolution |
| `--initial-subsidy` | `10` | LS-LMSR liquidity subsidy in USDC |
| `--fee-bps` | `100` | Market trading fee in bps |
| `--strategy-b` | — | Second strategy for Relative markets |

## `resolve-market`

```bash
bundie-sol resolve-market --market <pubkey>
```

Permissionless. Reverts if market is not Active or resolution slot not reached.

## `list-strategies`

```bash
bundie-sol list-strategies
```

Read-only. Returns a formatted table of every Strategy account on devnet:

```
address                                       | name                       | nav (USDC) | shares    | investors | fee
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
93amC41qp6YfTQwsugjgRG21eUtmbqiCx4VJjzE2xZYF  | Kamino Seed Strategy       |   1.043210 |   1000.00 |         3 | 500 bps

(1 strategies)
```

## `buy-shares`

See Participation commands above.

## `predict`

See Participation commands above.

## `nav`

```bash
bundie-sol nav --strategy <pubkey>
```

Read-only. Prints the strategy's current NAV, share price, TVL, investor count, and estimated APY.

---

# Demo sequences

## Two-agent creation-side walkthrough (Scene 2)

Terminal 1 (strategy-creator.sol):
```bash
bundie-sol --keypair ./strategy-creator.json create-strategy \
  --name "Stable Compounder" --protocol kamino --fee-bps 1000 --deposit 50
# Outputs: strategy: <SA>
```

Terminal 2 (market-maker.sol, different wallet):
```bash
# Watch for the new strategy to appear in list-strategies
bundie-sol list-strategies | grep "Stable Compounder"
# Then open a market on it
bundie-sol --keypair ./market-maker.json create-market \
  --strategy <SA> \
  --question "Will Stable Compounder exceed 8% APY in 7 days?" \
  --threshold-bps 800 \
  --resolution-days 7 \
  --initial-subsidy 10
```

The strategy-creator wallet is now blocked from taking positions on this market. The market-maker wallet can trade freely.

## End-to-end loop (for testing + demo seeding)

```bash
# 1. Create a strategy
bundie-sol create-strategy --name "E2E Test" --fee-bps 1000 --deposit 10
# -> <strategy>

# 2. Back it from a second wallet
bundie-sol --keypair ./backer.json buy-shares --strategy <strategy> --amount 25

# 3. Open a market from a third wallet
bundie-sol --keypair ./maker.json create-market \
  --strategy <strategy> --question "..." --threshold-bps 500 --resolution-days 1

# 4. Predict from a fourth wallet
bundie-sol --keypair ./predictor.json predict \
  --market <market> --side yes --amount 5

# 5. (one day later, past resolution slot) Resolve
bundie-sol resolve-market --market <market>
```

---

# Error cases

All commands print errors to stderr and exit with code 1.

| Error | Cause | Fix |
|---|---|---|
| `insufficient funds` | Wallet lacks USDC | Faucet at https://faucet.circle.com |
| `insufficient lamports` | Not enough SOL for fees | `solana airdrop 2 --url devnet` |
| `Account not found` | Strategy or market does not exist | Verify address |
| `CreatorCannotPredictOnOwnStrategy` | You are the strategy's creator | Use a different wallet |
| `WrongStrategyForMarket` | Passed a strategy that doesn't match the market | Pull the right strategy via `fetchMarket` / `list-strategies` |
| `InvalidStrategyAccount` | Strategy account discriminator mismatch | The account isn't a Bundie strategy |
| `MarketNotActive` | Market is paused or already resolved | Check market status |
| `ResolutionNotReached` | `resolve-market` called too early | Wait for `resolution_slot` |
| `InsufficientSnapshots` | NavOracle has no snapshots yet | `update_nav` must be called first (via the strategy-token program) |
| `amount below minimum deposit` | Deposit < strategy's `min_deposit` | Increase `--amount` |
| `name too long` | Strategy name > 32 chars | Shorten `--name` |
| `invalid public key input` | Malformed address | Double-check base58 |

---

# Seeded addresses (devnet)

Useful for immediate testing without creating accounts.

| Resource | Address |
|---|---|
| Kamino Seed Strategy | `93amC41qp6YfTQwsugjgRG21eUtmbqiCx4VJjzE2xZYF` |
| Seeded Prediction Market | `2XfBNnZ5YEH23tB4QdCNzXu2bJmeY5WwmMuikGgZP8wu` |

# Program addresses (devnet)

| Program | Address |
|---|---|
| Strategy Token Program (pinocchio) | `Y13kaQZ6NJgyfLiL5VjZ9k5QaFJnw4REM4A5Gsfg9VV` |
| Prediction Market Program (Anchor) | `Y13kynHKA6nfgDtYReVTuPZEVki6NmY9dYDihQT8j7i` |
| Devnet USDC Mint | `4zMMC9srt5Ri5X14GAgXhaHii3GnPAEERYPJgZJDncDU` |

# Devnet resources

| Resource | Link |
|---|---|
| Devnet USDC faucet | https://faucet.circle.com |
| Devnet SOL faucet | https://faucet.solana.com |
| Devnet explorer | https://explorer.solana.com/?cluster=devnet |

---

# Related surfaces

- **Webapp (`app.bundie.fi`)** — human surface. Backers and predictors use this; it wraps as a Seeker TWA for dApp Store distribution. Does not expose strategy or market creation.
- **EVM sibling (`@bundie/evm-mcp`)** — Bundie's EVM yield routing MCP, pre-existing. Different product (vault routing, not strategy composition). Connected via `mcp.bundie.fi/evm`.
- **Solana MCP** — deferred to post-hackathon. CLI + this skills file cover both Mode 1 and Mode 2 for MVP; MCP is a UX polish for Claude Desktop users, not a new capability.

# License

MIT.
