---
name: rwa-trade
description: >
  Buy and sell tokenized stocks & ETFs on Solana via the rwa CLI.
  Use when the user wants to trade, buy, sell, preview a quote, check market
  hours, or list tokens. Triggers: "buy TSLA", "sell stocks", "trade RWA",
  "quote NVDA", "market hours", "list tokens", "what can I buy on Solana".
---

# RWA Trade

Buy/sell 264 tokenized stocks & ETFs (Ondo Global Markets) on Solana. Always pass `--json`; add `-y` to execute, `--dry-run` to preview.

## Golden rules

- **NEVER use `&`, background, or parallel processes** — Jupiter rejects concurrent requests from one wallet (HTTP 400). One command at a time, `sleep 3` between sequential calls.
- For multi-token trades use the built-in `--parallel` flag (safe internal concurrency), NOT shell `&`.
- No `quote` command — preview with `--dry-run`.
- `--quote-only` (buy) quotes any size without a balance check; never executes (rejects `-y`), still enforces market hours + slippage. JSON `status` is `dry_run`. Use to size a buy before funding.
- `sell` swaps tokens → USDC; `send` transfers assets out. Never confuse them.
- CLI auto-retries transient swap failures. **Do NOT retry manually** unless it reports failure after retries.
- Multi-token = `buy-basket` / `sell-basket` / `close-all`. Never loop `buy`/`sell` by hand.
- Bulk filtering = `gm search` flags. **Never** pipe `gm list` through ad-hoc Python.

## Commands

```bash
rwa --json gm hours                                              # session + tradable_count
rwa --json gm search --tradable-only --sector Technology --type stock   # bulk scan
rwa --json gm tradable TSLA NVDA                                 # check specific symbols
rwa --json gm buy  TSLA 100 --dry-run                            # preview (100 USDC in)
rwa --json gm buy  TSLA 100 -y --slippage 50                     # execute, max 0.5% slippage
rwa --json gm sell TSLA all  -y                                  # sell all (also 50% / exact)
rwa --json gm buy-basket  AAPL 50 TSLA 50 NVDA 50 -y --parallel  # SYMBOL AMOUNT pairs
rwa --json gm sell-basket SPY 5 TSLA 3 NVDA all  -y --parallel
rwa --json gm close-all -y --parallel                            # sell ALL positions (~22s)
rwa --json gm close-all 50% -y                                   # sell 50% of every position
rwa --json gm reclaim                                            # close empty accounts, reclaim rent
```

- `close-all` skips positions < $1.50 (MMs reject tiny swaps) and lists them separately.
- Slippage: default 100 bps; hard-blocked above 3%. Amounts: exact `100`, `50%`, or `all`.

## Key JSON

```json
// buy -y / --dry-run (status "success" | "dry_run")
{"status":"success","amount":"0.258","token":"TSLAon","counter_amount":"100.00","counter_token":"USDC","tx":"https://solscan.io/tx/...","slippage_pct":-0.39}
// buy-basket → bought[]+total_usdc_spent · sell-basket → sold[]+total_usdc_received · close-all → sold[]+total_usdc
{"status":"success","sold":[{"token":"TSLAon","amount":"0.25","usdc":"96.50","tx":"..."}],"failed":[],"total_usdc":"96.50"}
```

`status:"success"` only means the run finished — **always check `failed[]`** (always present). `sold[]`/`bought[]` hold the swaps that worked; partial success is normal.

## Errors → action

| Error / error_kind | Agent action |
|---|---|
| `market_closed` | Run `gm hours`; tell user when it reopens |
| `not_tradable` | Skip token; verify with `gm tradable <SYM>` |
| `slippage_too_high` | Reduce size or skip; MM cooldown after rapid buy+sell — wait 30–60s |
| `Minimum buy amount is 1.0 USDC` | Use ≥ $1 |
| `Insufficient SOL for fees` | CLI does NOT auto-convert USDC→SOL — tell user to fund the wallet with SOL |
| `Insufficient USDC` | Tell user to fund USDC |
| `Balance is 0` | No position to sell |
| Swap failed code -1000/-2003/-2004/-2005 | CLI already auto-retried — do NOT retry manually |
| `No swap route` / HTTP 400 / HTTP 429 | Almost always parallel `&` — STOP, run sequentially |
| RPC `unavailable` / `all RPC endpoints failed` | Wait a few seconds; on repeats set `RWA_RPC_URL` to a dedicated endpoint (free key: Alchemy/Helius/QuickNode) |

## Overnight liquidity

Outside Regular Market (9:30 AM–4 PM ET) many tokens are illiquid. Usually fine: LLY, NVO, JNJ, PFE, ABBV, MRK. Often fail: AMGN, VRTX, UNH, ABT (trade in Regular Market). On `-2004`/`-2005`, substitute a token or wait — don't hammer the same one.

## Safe pre-trade flow

```bash
rwa keys show && rwa --json gm portfolio && rwa --json gm hours   # wallet, funds, market open?
rwa --json gm buy TSLA 100 -y
```
