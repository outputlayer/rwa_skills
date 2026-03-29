---
name: rwa-trade
description: >
  Buy and sell tokenized stocks & ETFs on Solana via the rwa CLI.
  Triggers: "buy TSLA", "sell AAPL", "market hours", "list tokens",
  "is TSLA tradable", "bulk buy", "basket", "close all positions",
  "sell everything", "dry run".
---

# Rules

- Always use `rwa --json` for agent calls
- Use `-y` only for real execution, never for `--dry-run`
- Never run wallet-changing commands in parallel
- Sleep `3` seconds between consecutive buy/sell/close-all/send commands
- There is no `quote` command: use `buy/sell --dry-run`
- For many sells, prefer `close-all` over manual loops
- Do not manually retry swap failures; the CLI already retries

# Shortest path

- User gave exact symbol + amount: run `buy` or `sell` directly
- User wants a preview: run `buy/sell --dry-run`
- User asks if one token is tradable: use `list --search <SYM>` not `hours --tradable`
- User asks what is tradable now: use `hours --tradable`
- User asks what to sell: use `portfolio` first, then sell/close-all
- User wants to exit everything: `close-all` -> `reclaim` -> `send`

# Core commands

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

Best practice:

- If the user already knows symbols and amounts, skip discovery calls
- For 2-3 small known buys, execute sequentially
- For larger or uncertain buys, dry-run each one first
- Never use `&`; use sequential commands only

Example execute:

```bash
rwa --json gm buy TSLA 100 -y
sleep 3
rwa --json gm buy AAPL 150 -y
sleep 3
rwa --json gm buy NVDA 200 -y
```

Example preview-first:

```bash
rwa --json gm buy TSLA 100 --dry-run
rwa --json gm buy AAPL 150 --dry-run
rwa --json gm buy NVDA 200 --dry-run
```

For sector/theme discovery:

1. `rwa --json gm list --search <theme>`
2. choose symbols
3. buy sequentially

Do not call `hours --tradable` first unless the user explicitly asked what is tradable right now.

# Bulk sell

- Sell one token: `gm sell`
- Reduce every position: `gm close-all 25% -y`
- Exit everything: `gm close-all -y`
- After exit: `gm reclaim`

Exit workflow:

```bash
rwa --json gm close-all -y
rwa --json gm reclaim
rwa --json gm send USDC all <ADDR> -y
rwa --json gm send SOL all <ADDR> -y
```

If the user wants exact post-liquidation withdrawal, prefer the exact `total_usdc` from `close-all`.

# Dry-run policy

- Use `--dry-run` for large orders
- Use `--dry-run` when liquidity/tradability is uncertain
- Skip `--dry-run` for small, explicit, low-risk orders if the user clearly wants execution
- `--dry-run` is cheaper than a failed execution

# Error handling

| Error | Action |
|-------|--------|
| `not tradable in current session` | skip token or show `list --search <SYM>` |
| `market is closed` | tell user when trading reopens |
| `No swap route` / `HTTP 400` | likely parallel flow or no liquidity |
| `Slippage too high` | skip or reduce size |
| `Solana RPC unavailable` | wait 5s, retry; after repeated failures suggest `RWA_RPC_URL` |

# Token efficiency

| Need | Cheapest useful call |
|------|----------------------|
| one token status | `list --search TSLA` |
| market session | `hours` |
| preview a trade | `buy/sell --dry-run` |
| everything tradable now | `hours --tradable` |
| reduce many positions | `close-all <pct>` |
| full exit | `close-all` |

# Good patterns

- Large single order: `buy --dry-run` -> `buy -y`
- Fast small order: `buy -y`
- One-token tradability check: `list --search <SYM>`
- User asks "buy these 5 symbols": no discovery, just sequential dry-run or buy
- User asks "what can I buy in semiconductors": `list --search semiconductor`
