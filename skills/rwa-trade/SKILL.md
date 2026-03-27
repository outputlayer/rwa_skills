---
name: rwa-trade
description: >
  Buy and sell tokenized stocks & ETFs on Solana via the rwa CLI.
  Triggers: "buy TSLA", "sell AAPL", "get quote", "market hours",
  "list tokens", "close all positions", "sell everything", "reduce portfolio".
---

# Rules

- No `&` or parallel — Jupiter rejects concurrent requests. Sequential only, `sleep 3` between commands
- Always use `--json -y` flags
- Use `close-all` for multiple positions (never sell one-by-one)
- `send` transfers tokens to another wallet; `sell` swaps for USDC — different commands

# Commands

```bash
rwa --json gm hours                      # Session status + tradable count
rwa --json gm hours --tradable           # List all tradable symbols
rwa --json gm list --search <keyword>    # Search by symbol/name/sector

rwa --json gm quote TSLA 100             # Buy quote (amount mandatory)
rwa --json gm quote TSLA 5 --sell        # Sell quote

rwa --json gm buy TSLA 100 -y           # Buy with USDC
rwa --json gm sell TSLA all -y          # Sell position (exact, 50%, or all)
rwa --json gm close-all -y             # Sell ALL positions
rwa --json gm close-all 50% -y         # Sell 50% of every position
rwa --json gm reclaim                   # Close empty accounts, reclaim SOL rent
```

# Amounts

Exact: `100` | Percentage: `50%` | All: `all`

# Sell all + withdraw

```bash
rwa --json gm close-all -y              # Returns total_usdc
rwa --json gm reclaim                   # Reclaim SOL rent
rwa --json gm send USDC 83.30 <ADDR> -y # Use exact amount from close-all result
rwa --json gm send SOL all <ADDR> -y    # Send remaining SOL
```

# Errors

| Error | Action |
|-------|--------|
| "not tradable in current session" | Skip — check `hours --tradable` |
| "market closed" | Tell user when it opens |
| "Quote not available" | No liquidity — skip token |
| "HTTP 400" / "No swap route" | Running parallel — run sequentially |
| "Swap failed (code -XXXX)" | CLI auto-retries — do NOT retry |
| "Solana RPC unavailable" | Wait 5s, retry. After 3 fails: set `RWA_RPC_URL` |

# Token cost per response

| Command | ~Tokens |
|---------|---------|
| `hours` | 42 |
| `hours --tradable` | 439 |
| `quote` | 49 |
| `buy/sell` | 250 |
| `list --search` | 50–400 |
| `portfolio` | 41–300 |

# Efficient workflows

**Sector portfolio** — `hours` (42 tok) → `list --search <sector>` (~400 tok) → buy each sequentially

**Sell one** — `sell TSLA all -y` directly (auto-checks tradable, no pre-check needed)

**Full liquidation** — `close-all -y` → `reclaim` → `send USDC all` → `send SOL all`
