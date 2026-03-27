---
name: rwa-trade
description: >
  Buy and sell tokenized stocks & ETFs on Solana via the rwa CLI.
  Triggers: "buy TSLA", "sell stocks", "trade RWA", "quote NVDA",
  "market hours", "list tokens", "what can I buy", "close all positions",
  "sell everything", "reduce portfolio".
---

# Rules

- **NEVER** use `&` or background for any `rwa` command — Jupiter rejects parallel requests (HTTP 400)
- **ALWAYS** run one command at a time with `sleep 3` between them
- **ALWAYS** use `--json` and `-y` flags
- **ALWAYS** check `tradable` status before buy/sell via `rwa --json gm list --search <SYM>`
- **ALWAYS** use `close-all` to sell multiple positions — never sell one by one
- `send` transfers tokens to a wallet. `sell` swaps tokens for USDC via Jupiter. They are different.

# Pre-trade checklist

```bash
rwa --json gm hours                      # 1. Market open? (status must be "open")
rwa --json gm list --search <SYM>        # 2. Token tradable? (tradable must be true)
rwa --json gm portfolio                  # 3. Has USDC/SOL?
```

# Commands

```bash
# Market & tokens
rwa --json gm hours                      # Session status + tradable count
rwa --json gm hours --tradable           # + list of tradable symbols
rwa --json gm list --search <keyword>    # Search by symbol/name/sector (ALWAYS use --search)

# Quote (amount is mandatory)
rwa --json gm quote TSLA 100             # Buy quote: 100 USDC -> TSLA
rwa --json gm quote TSLA 5 --sell        # Sell quote: 5 TSLA -> USDC

# Trade
rwa --json gm buy TSLA 100 -y            # Buy with USDC
rwa --json gm sell TSLA all -y           # Sell entire position
rwa --json gm close-all -y              # Sell ALL positions (handles retries, skips non-tradable)
rwa --json gm close-all 50% -y          # Sell 50% of every position

# After selling
rwa --json gm reclaim                    # Close empty accounts, reclaim SOL rent
```

# Amounts

Exact: `100` | Percentage: `50%` | All: `all`

# Sell all + withdraw flow

```bash
rwa --json gm close-all -y               # Sells all, returns total_usdc
rwa --json gm reclaim                    # Reclaim SOL rent
rwa --json gm send USDC 83.30 <ADDR> -y  # Send exact amount from close-all result
rwa --json gm send SOL all <ADDR> -y     # Send remaining SOL (auto-reserves tx fee)
```

# Errors

| Error | Action |
|-------|--------|
| "not tradable in current session" | Skip, check `rwa --json gm hours --tradable` |
| "market closed" | Tell user when it opens |
| "Quote not available from market maker" | No liquidity, skip token |
| "HTTP 400" / "No swap route" | You're running parallel. Stop. Run sequentially |
| "Swap failed (code -2003/-2004/-2005/-1000)" | CLI auto-retries. Do NOT retry manually |
| "Solana RPC unavailable" | Wait 5s, retry. After 3 fails: set `RWA_RPC_URL` |
| "Slippage too high" | Try larger amount or wait |

# Token weight per response (~tokens)

| Command | ~Tokens | Notes |
|---------|---------|-------|
| `hours` | 42 | Cheapest check |
| `hours --tradable` | 439 | Large — avoid unless listing all tradable |
| `quote` | 49 | |
| `buy/sell` result | 250 | |
| `list --search` | 50-400 | Varies by result count |
| `portfolio` (empty) | 41 | |
| `portfolio` (with positions) | 100-300 | ~30 tokens per position |
| `history` | 50 | |
| `reclaim` | 30-80 | |

# Minimum-token workflows

**Build sector portfolio** (e.g. "buy tech stocks"):
```bash
rwa --json gm hours                         # 1. Confirm open (42 tok)
rwa --json gm list --search technology      # 2. Get tradable tech stocks (~400 tok)
# Pick symbols from result, buy sequentially:
rwa --json gm buy TSLA 50 -y && sleep 3 && rwa --json gm buy AAPL 50 -y
```

**Check + sell** (minimal):
```bash
rwa --json gm sell TSLA all -y              # Skip hours/list — sell auto-checks tradable
```

**Full liquidation** (3 commands):
```bash
rwa --json gm close-all -y && sleep 5 && rwa --json gm reclaim && sleep 3 && rwa --json gm send USDC all <ADDR> -y
```
