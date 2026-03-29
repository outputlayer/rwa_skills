---
name: rwa-trade
description: >
  Buy and sell tokenized stocks & ETFs on Solana via the rwa CLI.
  Triggers: "buy TSLA", "sell AAPL", "market hours", "list tokens",
  "is TSLA tradable", "bulk buy", "basket", "close all positions",
  "sell everything", "dry run".
---

# Hard rules

- Always use `rwa --json` for agent-driven calls
- Use `-y` only for real execution, never with `--dry-run`
- Never run buy/sell/send/close-all in parallel
- Sleep `3` seconds between consecutive state-changing commands
- There is no `quote` command; use `buy/sell --dry-run`
- Do not manually retry swap failures; the CLI already retries
- For many sells, prefer `close-all` over manual sell loops
- Preserve the user amount exactly; never round or "normalize" decimals yourself
- Prefer surfaced error kinds over brittle text matching when interpreting failures

# Intent -> command

| User intent | Preferred command |
|-------------|-------------------|
| market session | `rwa --json gm hours` |
| what is tradable right now | `rwa --json gm hours --tradable` |
| check one symbol | `rwa --json gm list --search TSLA` |
| preview buy | `rwa --json gm buy TSLA 100 --dry-run` |
| execute buy | `rwa --json gm buy TSLA 100 -y` |
| preview sell | `rwa --json gm sell TSLA 50% --dry-run` |
| execute sell | `rwa --json gm sell TSLA 50% -y` |
| reduce every position | `rwa --json gm close-all 25% -y` |
| exit everything | `rwa --json gm close-all -y` |
| reclaim rent after sells | `rwa --json gm reclaim` |

# Do

- If the user already gave symbol + amount, go straight to `buy` or `sell`
- Use `--dry-run` for large, uncertain, or user-visible previews
- Use `list --search <SYM>` for one-token tradability checks
- Use `portfolio` first when the user asks what to sell
- Use `close-all` when the user wants to sell many positions
- Use `close-all --dry-run` when previewing a full or partial basket exit

# Don't

- Do not call `hours --tradable` just to check one symbol
- Do not call `list` before an already-specified buy unless discovery is needed
- Do not run `&`, `xargs -P`, or parallel command groups
- Do not manually round amounts
- Do not replace `close-all` with a manual sell loop unless the user asked for per-token control

# Commands

```bash
rwa --json gm hours
rwa --json gm hours --tradable
rwa --json gm list --search <keyword>

rwa --json gm buy TSLA 100 --dry-run
rwa --json gm buy TSLA 100 -y
rwa --json gm sell TSLA 50% --dry-run
rwa --json gm sell TSLA 50% -y
rwa --json gm close-all --dry-run
rwa --json gm close-all -y
rwa --json gm close-all 50% -y
rwa --json gm reclaim
```

# Amounts

Exact: `100`

Percentage: `50%`

All: `all`

Inputs with too many decimal places are rejected. Do not round manually.

# Bulk buy

Use this when the user already knows the basket:

1. skip discovery calls
2. optionally dry-run each order
3. buy sequentially
4. sleep `3` seconds between buys

Execute directly:

```bash
rwa --json gm buy TSLA 100 -y
sleep 3
rwa --json gm buy AAPL 150 -y
sleep 3
rwa --json gm buy NVDA 200 -y
```

Preview-first:

```bash
rwa --json gm buy TSLA 100 --dry-run
rwa --json gm buy AAPL 150 --dry-run
rwa --json gm buy NVDA 200 --dry-run
```

Use discovery only when needed:

1. `rwa --json gm list --search <theme>`
2. choose symbols
3. buy sequentially

# Bulk sell

Use this order of preference:

1. one token -> `gm sell`
2. reduce everything -> `gm close-all <pct>`
3. full exit -> `gm close-all`

Canonical full-exit workflow:

```bash
rwa --json gm close-all -y
rwa --json gm reclaim
rwa --json gm send USDC all <ADDR> -y
rwa --json gm send SOL all <ADDR> -y
```

If the user wants exact USDC withdrawal after liquidation, prefer the exact `total_usdc` from `close-all`.

# Dry-run policy

- Use `--dry-run` for large orders
- Use `--dry-run` when tradability or liquidity is uncertain
- Skip `--dry-run` for small explicit orders when the user clearly wants execution
- `--dry-run` is cheaper than a failed execution
- After a real `close-all`, prefer returned `total_usdc` over any preview estimate

# Error recovery

| Error | Action |
|-------|--------|
| `not_tradable` | skip token or show `list --search <SYM>` |
| `market_closed` | tell user when trading reopens |
| `No swap route` / `HTTP 400` | likely no liquidity or accidental parallel flow |
| `slippage_too_high` | reduce size or skip token |
| `Solana RPC unavailable` | wait 5s, retry; after repeated failures suggest `RWA_RPC_URL` |

# Cheapest useful call

| Need | Preferred call |
|------|----------------|
| one token status | `list --search TSLA` |
| market session | `hours` |
| all tradable now | `hours --tradable` |
| preview one trade | `buy/sell --dry-run` |
| reduce many positions | `close-all <pct>` |
| full exit | `close-all` |

# Canonical patterns

- Large single order: `buy --dry-run` -> `buy -y`
- Fast small order: `buy -y`
- One-token tradability check: `list --search <SYM>`
- User says "buy these 5 symbols": no discovery; dry-run or buy sequentially
- User says "what can I buy in semiconductors": `list --search semiconductor`
