---
name: rwa-trade
description: >
  Buy and sell tokenized stocks & ETFs on Solana via the rwa CLI.
  Use when user wants to trade, buy, sell, get a quote, check market hours,
  or list available tokens. Triggers: "buy TSLA", "sell stocks", "trade RWA",
  "quote NVDA", "market hours", "list tokens", "what can I buy on Solana".
---

# RWA Trade

Buy and sell 264 tokenized stocks & ETFs ([Ondo Global Markets](https://ondo.finance/)) on Solana.

## Prerequisites

Assume `rwa` is installed and in PATH. If "command not found", install:
```bash
curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh | bash
```

Do NOT prepend `export PATH=...` to every command. The installer adds rwa to PATH automatically.

## Agent Guidelines

- **NEVER use `&` (background) for ANY rwa command** — not for quotes, trades, history, or anything else. Jupiter API rejects concurrent requests from the same wallet with HTTP 400. Always run one command at a time, sequentially.
- **Wait between commands**: Add `sleep 3` between consecutive rwa commands. Solana RPC rate-limits aggressively.
- **RPC errors ("Solana RPC unavailable")**: Wait at least 5 seconds before retrying. Do NOT retry immediately. After 3 failures, stop and tell the user to set `RWA_RPC_URL`.
- **Quote requires AMOUNT**: `rwa --json gm quote <SYMBOL> <AMOUNT>`. Amount is mandatory (USDC for buy, tokens for sell).
- **Batch quotes**: Use a sequential loop (NO `&`). Amount is required:
  ```bash
  for sym in TSLA AAPL NVDA SPY; do rwa --json gm quote $sym 100 2>/dev/null; sleep 2; done
  ```
- **Batch history**: Sequential loop (NO `&`):
  ```bash
  for sym in TSLA AAPL NVDA; do rwa --json gm history $sym -r 1W 2>/dev/null; sleep 2; done
  ```
- **Find tokens**: Use `rwa --json gm list --search <keyword>`. NEVER dump full list without `--search`.
- **Buy multiple**: Sequential with sleep:
  ```bash
  rwa gm buy TSLA 100 -y && sleep 5 && rwa gm buy AAPL 100 -y && sleep 5 && rwa gm buy NVDA 100 -y
  ```
- **Sell multiple**: Use `close-all` to sell everything at once:
  ```bash
  rwa --json gm close-all -y
  ```
  This sells every GM position sequentially, skipping any that fail. Do NOT sell each token manually.
- **Reduce all positions**: Use `close-all` with a percentage:
  ```bash
  rwa --json gm close-all 50% -y    # Sell 50% of every position
  rwa --json gm close-all 10% -y    # Sell 10% of every position
  ```

## Flags

| Flag | Required for agents |
|------|:---:|
| `--json` | Yes — always use for machine-readable output |
| `-y` | Yes — skip confirmation on buy/sell |
| `--slippage <BPS>` | No — max slippage in basis points (e.g. 50 = 0.5%). Default: 100 (1%). Hard block at 3% |
| `--rpc-url <URL>` | No — custom Solana RPC (or `RWA_RPC_URL` env) |

## Workflow

### 1. Check Market Hours

Trading is 24/5: Sunday 8 PM — Friday 8 PM ET.

```bash
rwa --json gm hours
```

### 2. Find Tokens

**ALWAYS use `--search` to filter** — never dump the full list, it wastes tokens:
```bash
rwa --json gm list --search healthcare  # Search by sector
rwa --json gm list --search technology  # Search by sector
rwa --json gm list --search energy      # Search by sector
rwa --json gm list --search etf         # Search by type
rwa --json gm list --search biotech     # Search by keyword
rwa --json gm list --search amgen       # Search by company name
```

Only use `rwa --json gm list` (no filter) if the user explicitly asks for all tokens.
JSON output includes `type` ("stock" or "etf"), `sector` (e.g. "Technology", "Healthcare"), cleaned company name, and `tradable` (true/false for current session).
Both `TSLA` and `TSLAon` symbol formats accepted.

Available sectors: Technology, Healthcare, Financials, Consumer Discretionary, Energy, Industrials, Materials, Utilities, Real Estate Sector, Infrastructure.

### 3. Get a Quote

```bash
rwa --json gm quote TSLA 100          # Buy: 100 USDC → TSLA
rwa --json gm quote TSLA 5 --sell     # Sell: 5 TSLA → USDC
```

### 4. Execute Trade

```bash
rwa gm buy TSLA 100 -y       # Buy with USDC
rwa gm buy TSLA 100 -y --slippage 50  # Buy with max 0.5% slippage
rwa gm sell TSLA all -y       # Sell entire position
rwa gm sell SPY 50% -y        # Sell half
rwa --json gm close-all -y    # Sell ALL positions at once
rwa --json gm close-all 50% -y # Sell 50% of every position
rwa --json gm close-all 10% -y # Sell 10% of every position
```

### 5. Reclaim Rent

Close empty token accounts to reclaim SOL rent (~0.002 SOL per account):

```bash
rwa --json gm reclaim                # Close all empty token accounts
rwa --json gm reclaim --token TSLA   # Close only empty TSLA accounts
```

JSON output:
```json
{"status":"success","accounts_closed":5,"sol_reclaimed":"0.010196400","signatures":["5K1z..."]}
```

Note: USDC token account is preserved (never closed) since it's needed for trading.

### 6. Close All / Reduce All Positions

Use `close-all` to sell every GM token position. Optionally pass a percentage to sell only a portion of each position.

```bash
rwa --json gm close-all -y          # Sell 100% of every position
rwa --json gm close-all 50% -y      # Sell 50% of every position
rwa --json gm close-all 10% -y      # Sell 10% of every position
```

JSON output includes `sold` (successful sells) and `failed` (skipped tokens):
```json
{"status":"success","sold":[{"token":"TSLAon","amount":"0.25","usdc":"96.50","tx":"https://solscan.io/tx/..."}],"failed":[],"total_usdc":"96.50"}
```

**ALWAYS use `close-all` instead of selling tokens one by one.** The CLI handles retries, delays, and error skipping internally. Positions < $1.50 are skipped (market makers reject small swaps).

- "Sell all positions" → `rwa --json gm close-all -y`
- "Reduce portfolio by 50%" → `rwa --json gm close-all 50% -y`  
- "Take 10% profit" → `rwa --json gm close-all 10% -y`

## Amount Formats

- Exact: `100` (USDC for buy, tokens for sell)
- Percentage: `50%`
- All: `all`

## Key JSON Output

```json
// rwa --json gm hours
{"status":"open","session":"Regular Market","session_hours":"9:30 AM – 3:59 PM ET","now":"Monday 10:30 AM ET","countdown":"next session in 5h 30m","tradable_count":214}

// rwa --json gm quote TSLA 100
{"input":"100","input_token":"USDC","output":"0.025844","output_token":"TSLAon","input_usd":100.0,"output_usd":99.6,"slippage_pct":-0.39,"price_impact_pct":-0.39,"fee_bps":10,"tradable":true}

// rwa --json gm list --search tsla
[{"symbol":"TSLAon","name":"Tesla","type":"stock","sector":"Consumer Discretionary","tradable":true}]

// rwa --json gm buy TSLA 100 -y
{"status":"success","amount":"0.258","token":"TSLAon","counter_amount":"100.00","counter_token":"USDC","tx":"https://solscan.io/tx/...","slippage_pct":-0.39}
```

## Errors & What To Do

The CLI has built-in retry logic (max 2 retries with fresh orders). Most transient errors resolve automatically. **Do NOT manually retry** unless the CLI says the swap failed after retries.

| Error | Cause | Agent action |
|-------|-------|-------------|
| "market closed" | Trading hours ended | Run `rwa --json gm hours`, tell user when it opens |
| "Solana RPC unavailable" | Rate-limited or down | Wait 5s, retry once. After 3 failures, tell user to set `RWA_RPC_URL` |
| "No wallet found" | No keypair | Run `rwa keys generate` |
| "Insufficient SOL for gas" | SOL < 0.01 | CLI auto-topups from USDC. If still fails, user needs to send SOL |
| "Insufficient USDC" | Not enough to buy | Tell user to send USDC to wallet |
| "Balance is 0" | Nothing to sell | Tell user they have no position |
| "Minimum buy amount is 1.0 USDC" | Amount too small | Use at least $1 |
| "Slippage too high" | Slippage >3% on quote | Normal for small sells (<$1.50). Skip this trade or try larger amount |
| "Swap failed (code -2003)" | Quote expired between order and execute | CLI auto-retries with fresh order. Do NOT retry manually |
| "Swap failed (code -2004)" | Market maker rejected | CLI auto-retries with fresh order. Do NOT retry manually |
| "Swap failed (code -2005)" | Jupiter internal error | CLI auto-retries with fresh order. Do NOT retry manually |
| "Swap failed (code -1000)" | Transaction didn't land | CLI auto-retries. Do NOT retry manually |
| "No swap route found" | Running commands in parallel with `&` | **STOP.** Run sequentially, one at a time |
| "Jupiter API error (HTTP 400)" | Parallel requests or no liquidity | Stop using `&`. If sequential, token has no liquidity |
| "Jupiter API error (HTTP 429)" | Rate limited | Wait 5s, retry. Never run parallel commands |
| "Quote not available from market maker" | No liquidity for this token | Skip this token, try another |

## Safety — Pre-Trade Checklist

The CLI enforces these checks automatically, but agents should verify upfront to give clear guidance:

1. **Wallet exists**: `rwa keys show` — if fails, run `rwa keys generate` first
2. **Has SOL for gas**: portfolio must show `sol >= 0.005`
3. **Has USDC for buys**: portfolio must show `usdc >= buy amount`
4. **Market is open**: `rwa --json gm hours` must return `"status":"open"`

### CRITICAL: send vs sell

- `rwa gm sell` — sells tokens for USDC (swap via Jupiter)
- `rwa gm send` — transfers tokens/USDC/SOL to another wallet (no swap)
- `send USDC all` sends **ALL USDC** in the wallet, not just recent sell proceeds
- **When user says "sell X% and send proceeds"**: calculate the USDC amount from sell results and use `send USDC <exact_amount>`, NEVER `send USDC all`
- **To sell all and send**: use `rwa --json gm close-all -y`, then `send USDC <amount_from_close_all_total_usdc>`
- **To reduce all positions by X%**: use `rwa --json gm close-all X% -y` (e.g. `close-all 50% -y`)

**Recommended agent flow before any trade:**
```bash
# Step 1: Verify wallet
rwa keys show

# Step 2: Check balances
rwa --json gm portfolio

# Step 3: Check market
rwa --json gm hours

# Step 4: Only then trade
rwa gm buy TSLA 100 -y

# Step 5: After selling all, reclaim rent
rwa --json gm reclaim
```

If any step fails, tell the user what's missing before attempting the trade.
