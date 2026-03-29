---
name: rwa-trade
description: >
  Buy and sell tokenized stocks & ETFs on Solana via the rwa CLI.
  Triggers: "buy TSLA", "sell AAPL", "preview trade", "market hours",
  "list tokens", "close all positions", "sell everything", "reduce portfolio",
  "is TSLA tradable", "dry run".
---

# Rules

- **NEVER run commands in parallel (`&`)** — Jupiter rejects concurrent requests from the same wallet
- Always use `--json -y` flags for buy/sell/send
- `sleep 3` between consecutive commands
- Use `close-all` for multiple sells (never sell one-by-one manually)
- `send` transfers tokens to another wallet; `sell` swaps for USDC — different commands
- CLI auto-retries swap errors (max 2×) — do NOT retry manually

# Commands

```bash
rwa --json gm hours                       # Session status + tradable count
rwa --json gm hours --tradable            # List all tradable symbols now
rwa --json gm list --search <keyword>     # Search by symbol / name / sector

rwa --json gm buy TSLA 100 -y            # Buy with USDC
rwa --json gm buy TSLA 100 --dry-run     # Preview: validate + quote, no execution
rwa --json gm sell TSLA all -y           # Sell (exact, 50%, or all)
rwa --json gm sell TSLA all --dry-run    # Preview sell
rwa --json gm close-all -y              # Sell ALL positions sequentially
rwa --json gm close-all 50% -y          # Sell 50% of every position
rwa --json gm close-all --dry-run       # Preview what would be sold
rwa --json gm reclaim                    # Close empty accounts, reclaim SOL rent
```

# --dry-run output (JSON)

`buy/sell --dry-run` validates balance + tradability and returns the full quote without executing:

```json
{
  "status": "dry_run",
  "amount": "1.234567890",
  "token": "TSLAon",
  "counter_amount": "100.00",
  "counter_token": "USDC",
  "tx": "",
  "slippage_pct": 0.1234,
  "price_impact_pct": 0.0012,
  "fee_bps": 25,
  "gasless": true,
  "router": "jupiterz"
}
```

# Amounts

Exact: `100` | Percentage: `50%` | All: `all`

# Buying multiple tokens (basket)

No multi-buy command — run sequentially with `&&` (stops chain on failure):

```bash
rwa --json gm buy TSLA 100 -y && sleep 3 && \
rwa --json gm buy AAPL 150 -y && sleep 3 && \
rwa --json gm buy NVDA 200 -y
```

Preview all first (no execution, no `-y` needed):
```bash
rwa --json gm buy TSLA 100 --dry-run && \
rwa --json gm buy AAPL 150 --dry-run && \
rwa --json gm buy NVDA 200 --dry-run
```

# Sell all + withdraw

```bash
rwa --json gm close-all -y               # Returns total_usdc
rwa --json gm reclaim                    # Reclaim SOL rent (~0.002 SOL per empty account)
rwa --json gm send USDC 83.30 <ADDR> -y  # Use EXACT amount from close-all result
rwa --json gm send SOL all <ADDR> -y     # Send remaining SOL
```

# Errors

| Error | Action |
|-------|--------|
| "not tradable in current session" | Skip — check `hours --tradable` |
| "market closed" | Tell user when it opens |
| "Quote not available" | No liquidity — skip token |
| "HTTP 400" / "No swap route" | Running parallel — run sequentially |
| "Swap failed (code -XXXX)" | CLI auto-retries — do NOT retry manually |
| "Solana RPC unavailable" | Wait 5s, retry. After 3 fails: ask user to set `RWA_RPC_URL` |
| ">3% slippage" | Hard-blocked by CLI — token illiquid, skip or try smaller amount |

# Token cost per response

| Command | ~Tokens |
|---------|---------|
| `hours` | 42 |
| `hours --tradable` | 439 |
| `buy/sell --dry-run` | 55 |
| `buy/sell` (executed) | 250 |
| `close-all` | 150–400 |
| `list --search` | 50–400 |
| `portfolio` | 41–300 |

# Efficient workflows

**Check before large order** — `buy TSLA 500 --dry-run` (55 tok) → if ok, `buy TSLA 500 -y` (250 tok)

**Skip dry-run for small confident orders** — `buy TSLA 100 -y` directly saves 55 tokens

**Sector portfolio** — `hours` (42 tok) → `list --search Technology` (~400 tok) → buy each sequentially

**Full liquidation** — `close-all -y` → `reclaim` → `send USDC all` → `send SOL all`
