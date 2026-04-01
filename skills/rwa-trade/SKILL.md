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

## ⚡ Buying multiple tokens — the fast path

**One `list` call gives you tradable status for ALL tokens. Then one `buy-basket` buys them all.**

```bash
# Step 1: check tradable + filter — ONE RPC call
rwa --json gm list | python3 -c "
import sys, json
tokens = json.load(sys.stdin)
tradable = [t['symbol'] for t in tokens if t['tradable']]
print('Tradable:', tradable[:10])  # or filter by sector
"

# Step 2: buy everything in one command, parallel
rwa --json gm buy-basket AAPL 50 TSLA 50 NVDA 50 SPY 50 --parallel -y
```

**Never do this (wastes tokens, slow, breaks at scale):**

```bash
# ❌ BAD — one check per token
rwa --json gm list --search AAPL
rwa --json gm list --search TSLA
rwa --json gm list --search NVDA
rwa gm buy AAPL 50 -y && sleep 5 && rwa gm buy TSLA 50 -y && sleep 5 && rwa gm buy NVDA 50 -y
```

### Full "build a themed portfolio" in 3 commands

```bash
# 1. Check USDC balance
rwa --json gm portfolio

# 2. Get all tradable tokens + filter by sector — ONE call
rwa --json gm list | python3 -c "
import sys, json
tokens = [t for t in json.load(sys.stdin) if t.get('sector')=='Healthcare' and t['tradable']]
pairs = ' '.join(f\"{t['symbol']} 25\" for t in tokens[:8])
print(pairs)
"
# Output example: JNJon 25 LLYon 25 PFEon 25 ABBVon 25 MRKon 25

# 3. Buy all at once (paste the output from step 2)
rwa --json gm buy-basket JNJon 25 LLYon 25 PFEon 25 ABBVon 25 MRKon 25 --parallel -y
```

`buy-basket` validates USDC balance and checks tradability before the first swap.
Tokens that fail go into `failed[]` — the rest still execute. Never loop `buy` manually.

### Full rebalance — sell everything, rebuy a new set

```bash
# 1. Sell ALL current positions
rwa --json gm close-all --parallel -y

# 2. Find new tradable tokens
rwa --json gm list | python3 -c "import sys,json; tokens=[t for t in json.load(sys.stdin) if t.get('sector')=='Technology' and t['tradable']]; [print(t['symbol']) for t in tokens[:6]]"

# 3. Buy new basket in one command
rwa --json gm buy-basket AAPL 100 NVDA 100 MSFT 100 --parallel -y
```

---

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
- **Check tradable status**: Use `rwa --json gm list` ONCE and filter with `python3`. The `tradable` field is per-token per-session. Do NOT query tokens one-by-one.
- **Batch previews**: Use a sequential loop (NO `&`). Amount is required:

  ```bash
  for sym in TSLA AAPL NVDA SPY; do rwa --json gm buy $sym 100 --dry-run 2>/dev/null; sleep 2; done
  ```

- **Batch history**: Sequential loop (NO `&`):

  ```bash
  for sym in TSLA AAPL NVDA; do rwa --json gm history $sym -r 1W 2>/dev/null; sleep 2; done
  ```

- **Find one token**: Use `rwa --json gm list --search <keyword>` for a single symbol or name.
- **Find tokens by sector/type**: Use `rwa --json gm list` piped to `python3` for filtering — this is ONE call instead of N separate searches:

  ```bash
  rwa --json gm list | python3 -c "import sys,json; [print(t['symbol'],t['name']) for t in json.load(sys.stdin) if t.get('sector')=='Healthcare' and t['tradable']]"
  ```

  Filter by sector (Healthcare, Technology, Financials, Energy, etc.), type (stock/etf), or tradable status — all from one call.
- **Buy multiple tokens**: Use `buy-basket` with `SYMBOL AMOUNT` pairs. Each token can have a **different** USDC amount:

  ```bash
  rwa --json gm buy-basket TSLA 100 AAPL 50 NVDA 25 -y --parallel
  ```

  Do NOT use the old `--usdc-each` flag — it no longer exists.

- **Sell multiple tokens**: Use `sell-basket` for partial sells, or `close-all` to sell everything:

  ```bash
  rwa --json gm sell-basket SPY 5 TSLA 3 NVDA all -y --parallel  # sell specific amounts
  rwa --json gm close-all -y                                      # sell 100% of everything
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
| `--parallel` | No — safe internal concurrency for `close-all` and `buy-basket`. NOT the same as shell `&` |
| `--slippage <BPS>` | No — max slippage in basis points (e.g. 50 = 0.5%). Default: 100 (1%). Hard block at 3% |
| `--rpc-url <URL>` | No — custom Solana RPC (or `RWA_RPC_URL` env) |

## Workflow

### 1. Check Market Hours

Trading is 24/5: Sunday 8 PM — Friday 8 PM ET.

```bash
rwa --json gm hours
```

### 2. Find Tokens

**Single token**: `rwa --json gm list --search TSLA` — fast, one result.

**Multiple tokens or sector scan**: One `list` call + python3 filter is more efficient than N separate `--search` calls:

```bash
# All tradable healthcare stocks (1 RPC call vs 8+ searches)
rwa --json gm list | python3 -c "import sys,json; [print(json.dumps(t)) for t in json.load(sys.stdin) if t.get('sector')=='Healthcare' and t['tradable']]"

# All tradable ETFs
rwa --json gm list | python3 -c "import sys,json; [print(json.dumps(t)) for t in json.load(sys.stdin) if t.get('type')=='etf' and t['tradable']]"

# Count by sector
rwa --json gm list | python3 -c "import sys,json; from collections import Counter; c=Counter(t.get('sector','ETF') for t in json.load(sys.stdin) if t['tradable']); [print(f'{s}: {n}') for s,n in c.most_common()]"
```

Use `rwa --json gm list --search <keyword>` only for quick single-token lookups.
JSON output includes `type` ("stock" or "etf"), `sector` (e.g. "Technology", "Healthcare"), cleaned company name, and `tradable` (true/false for current session).
Both `TSLA` and `TSLAon` symbol formats accepted.

Available sectors: Technology, Healthcare, Financials, Consumer Discretionary, Energy, Industrials, Materials, Utilities, Real Estate Sector, Infrastructure.

### 3. Preview a Trade (Dry Run)

There is no separate `quote` command — use `--dry-run` on buy/sell.

```bash
rwa --json gm buy TSLA 100 --dry-run    # Buy preview: 100 USDC → TSLA
rwa --json gm sell TSLA 5 --dry-run     # Sell preview: 5 TSLA → USDC
```

### 4. Execute Trade

```bash
rwa gm buy TSLA 100 -y                  # Buy with USDC
rwa gm buy TSLA 100 -y --slippage 50   # Buy with max 0.5% slippage
rwa gm sell TSLA all -y                # Sell entire position
rwa gm sell SPY 50% -y                 # Sell half
rwa --json gm close-all -y             # Sell ALL positions (sequential)
rwa --json gm close-all -y --parallel  # Sell ALL positions in parallel (~22s flat)
rwa --json gm close-all 50% -y --parallel  # Sell 50% in parallel

# Buy a basket of tokens with per-token USDC amounts
rwa --json gm buy-basket JNJ 25 LLY 25 PFE 25 ABBV 25 -y --parallel
rwa --json gm buy-basket JNJ 25 LLY 25 PFE 25 ABBV 25 -y --dry-run    # Preview
rwa --json gm buy-basket AAPL 100 TSLA 50 NVDA 25 -y                   # Different amounts

# Sell a basket of tokens with per-token amounts
rwa --json gm sell-basket SPY 5 TSLA 3 NVDA all -y --parallel
rwa --json gm sell-basket SPY 5 TSLA 3 -y --dry-run                    # Preview
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

Use `close-all` to sell every GM token position. Add `--parallel` to run all sells simultaneously.

```bash
rwa --json gm close-all -y --parallel       # Sell 100% in parallel (~22s for any N tokens)
rwa --json gm close-all -y                  # Sell 100% sequentially (N×22s)
rwa --json gm close-all 50% -y --parallel   # Sell 50% of every position in parallel
rwa --json gm close-all 10% -y             # Sell 10% sequentially
```

JSON output always includes `sold`, `failed`, and `total_usdc`:

```json
{"status":"success","sold":[{"token":"TSLAon","amount":"0.25","usdc":"96.50","tx":"https://solscan.io/tx/..."}],"failed":[],"total_usdc":"96.50"}
```

`status:"success"` means the operation completed — check `failed[]` to see if any individual swaps failed. **`failed[]` is always present** even on partial success.

**ALWAYS use `close-all` instead of selling tokens one by one.** Positions < $1.50 are skipped (market makers reject small swaps).

- "Sell all positions" → `rwa --json gm close-all -y --parallel`
- "Reduce portfolio by 50%" → `rwa --json gm close-all 50% -y --parallel`
- "Take 10% profit" → `rwa --json gm close-all 10% -y`

### 7. Buy a Basket of Tokens

Buy multiple tokens with interleaved `SYMBOL AMOUNT` pairs — each token gets its own USDC allocation:

```bash
rwa --json gm buy-basket JNJ 25 LLY 25 PFE 25 ABBV 25 -y --parallel   # All same amount
rwa --json gm buy-basket AAPL 100 TSLA 50 NVDA 25 -y --parallel        # Different amounts
rwa --json gm buy-basket JNJ 25 LLY 25 -y --dry-run                    # Preview
rwa --json gm buy-basket JNJ 25 LLY 25 -y                              # Sequential
```

Syntax: `SYMBOL1 AMOUNT1 SYMBOL2 AMOUNT2 ...` — pairs, no `--usdc-each`.

JSON output:

```json
{"status":"success","bought":[{"token":"JNJon","received":"0.061","usdc":"25","tx":"https://solscan.io/tx/...","slippage_pct":-0.52}],"failed":[],"total_usdc_spent":"25.00"}
```

`failed[]` contains `{token, error}` for each failed swap — always check it. Partial success (some bought, some failed) still returns `status:"success"`.

### 8. Sell a Basket of Tokens

Sell multiple tokens with per-token amounts in one command:

```bash
rwa --json gm sell-basket SPY 5 TSLA 3 NVDA all -y --parallel   # Sell specific amounts
rwa --json gm sell-basket SPY 50% TSLA 50% -y --parallel        # Sell % of each
rwa --json gm sell-basket SPY 5 TSLA 3 -y --dry-run             # Preview
```

Syntax: `SYMBOL1 AMOUNT1 SYMBOL2 AMOUNT2 ...` — same pair format as `buy-basket`. Amounts can be exact tokens, `50%`, or `all`.

JSON output:

```json
{"status":"success","sold":[{"token":"SPYon","amount":"5.000000","usdc":"1316.25","tx":"https://solscan.io/tx/...","slippage_pct":-0.12}],"failed":[],"total_usdc_received":"1316.25"}
```

**When to use `sell-basket` vs `close-all`:**

- Use `sell-basket` when amounts per token differ or when selling only some positions
- Use `close-all` when selling all positions (or a uniform percentage of all)

## Amount Formats

- Exact: `100` (USDC for buy, tokens for sell)
- Percentage: `50%`
- All: `all`

## Key JSON Output

```json
// rwa --json gm hours
{"status":"open","session":"Regular Market","session_hours":"9:30 AM – 3:59 PM ET","now":"Monday 10:30 AM ET","countdown":"next session in 5h 30m","tradable_count":214}

// rwa --json gm buy TSLA 100 --dry-run
{"status":"dry_run","amount":"0.025844","token":"TSLAon","counter_amount":"100.00","counter_token":"USDC","slippage_pct":-0.39}

// rwa --json gm list --search tsla
[{"symbol":"TSLAon","name":"Tesla","type":"stock","sector":"Consumer Discretionary","tradable":true}]

// rwa --json gm buy TSLA 100 -y
{"status":"success","amount":"0.258","token":"TSLAon","counter_amount":"100.00","counter_token":"USDC","tx":"https://solscan.io/tx/...","slippage_pct":-0.39}

// rwa --json gm buy-basket JNJ 25 LLY 25 -y --parallel
{"status":"success","bought":[{"token":"JNJon","received":"0.061","usdc":"25","tx":"https://solscan.io/tx/...","slippage_pct":-0.52},{"token":"LLYon","received":"0.058","usdc":"25","tx":"https://solscan.io/tx/...","slippage_pct":-0.19}],"failed":[],"total_usdc_spent":"50.00"}

// rwa --json gm sell-basket SPY 5 TSLA 3 -y --parallel
{"status":"success","sold":[{"token":"SPYon","amount":"5.000000","usdc":"1316.25","tx":"https://solscan.io/tx/...","slippage_pct":-0.12},{"token":"TSLAon","amount":"3.000000","usdc":"1156.40","tx":"https://solscan.io/tx/...","slippage_pct":-0.08}],"failed":[],"total_usdc_received":"2472.65"}

// rwa --json gm close-all -y --parallel  (partial failure example)
{"status":"success","sold":[{"token":"JNJon","amount":"0.061","usdc":"14.95","tx":"https://solscan.io/tx/..."}],"failed":[{"token":"TSLAon","error":"Swap failed (code -2004): swap rejected"}],"total_usdc":"14.95"}
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
| `failed[]` contains `{token, error}` | buy-basket or close-all partial failure | Check `failed[].error` for each failed swap. Not all failed = fatal: `sold[]`/`bought[]` still contain successful results |
| `slippage_too_high` in `failed[].error` | MM cooldown (-10% quote blocked) | Can happen if same token was rapidly bought+sold multiple times. Wait 30–60s, retry with fresh order |
| `not_tradable` in `failed[].error` | Token not tradable in current session | Run `rwa --json gm hours`, skip this token until market opens |

## Portfolio Building (sector/strategy)

When user asks for a themed portfolio (pharma, tech, etc.):

1. Check balance: `rwa --json gm portfolio` — note available USDC
2. Check hours: `rwa --json gm hours` — verify tradable
3. Discover tokens in ONE call:

   ```bash
   rwa --json gm list | python3 -c "import sys,json; [print(json.dumps(t)) for t in json.load(sys.stdin) if t.get('sector')=='Healthcare' and t['tradable']]"
   ```

4. Build the portfolio with `buy-basket --parallel` (preferred) or sequential fallback:

   ```bash
   # Preferred: all buys in ~22s flat (each token gets its own USDC amount)
   rwa --json gm buy-basket JNJ 10 LLY 10 PFE 10 ABBV 10 -y --parallel

   # Different amounts per token
   rwa --json gm buy-basket JNJ 20 LLY 15 PFE 10 ABBV 10 -y --parallel

   # Fallback: sequential with sleep
   rwa --json gm buy JNJ 10 -y && sleep 3 && rwa --json gm buy LLY 10 -y && sleep 3 && rwa --json gm buy PFE 10 -y
   ```

5. If a buy fails (slippage/no route), it lands in `failed[]` — skip it and try next — do NOT retry immediately
6. Verify: `rwa --json gm portfolio`

**Overnight liquidity warning**: Many tokens have poor liquidity outside Regular Market (9:30 AM – 4 PM ET). If buys fail with `code -2004`/`-2005`, do NOT retry the same token immediately — the MM cooldown will quote -10%, which our guard blocks (goes to `failed[]` instead of executing). Substitute a different token or wait for Regular Market.

- **Liquid overnight** (usually work): `LLY`, `NVO`, `JNJ`, `PFE`, `ABBV`, `MRK`
- **Illiquid overnight** (often fail): `AMGN`, `VRTX`, `UNH`, `ABT` — buy these during Regular Market only

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
