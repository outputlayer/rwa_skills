---
name: rwa-trade
description: >
  Buy and sell tokenized stocks & ETFs on Solana via the rwa CLI.
  Use when the user wants to trade, buy, sell, preview a quote, check market
  hours, or list tokens. Triggers: "buy TSLA", "sell stocks", "trade RWA",
  "quote NVDA", "market hours", "list tokens", "what can I buy on Solana".
---

# RWA Trade

Buy/sell 438 tokenized stocks & ETFs (Ondo Global Markets) on Solana. Always pass `--json`; add `-y` to execute, `--dry-run` to preview.

## Golden rules

- **NEVER use `&`, background, or parallel shell processes** — one `rwa` process at a time (a lock enforces it). Multi-token commands parallelize internally.
- Minimum buy: **5 USDC** (single buy and per basket item).
- No `quote` command — preview with `--dry-run`.
- `--quote-only` (buy) quotes any size without a balance check; never executes (rejects `-y`). JSON `status` is `dry_run`. Use to size a buy before funding.
- `sell` swaps tokens → USDC; `send` transfers assets out. Never confuse them.
- CLI auto-retries transient swap failures. **Do NOT retry manually** unless it reports failure after retries.
- Multi-token = `buy-basket` / `sell-basket` / `close-all` — they run **in parallel by default** (bounded internally). Never loop `buy`/`sell` by hand. `--sequential` is a rate-limit fallback only.
- Bulk filtering = `gm search` flags. **Never** pipe `gm list` through ad-hoc Python.

## Commands

```bash
rwa --json gm hours                                          # session + tradable_count
rwa --json gm search --tradable-only --sector Technology     # bulk scan
rwa --json gm search --tag asia --tag "fixed income"         # any Ondo tag: region, asset class, factor
rwa --json gm tradable TSLA NVDA                             # check specific symbols
rwa --json gm buy  TSLA 100 --dry-run                        # preview (100 USDC in)
rwa --json gm buy  TSLA 100 -y --slippage 50                 # execute, max 0.5% slippage
rwa --json gm sell TSLA all  -y                              # sell all (also 50% / exact)
rwa --json gm buy-basket  AAPL 50 TSLA 50 NVDA 50 -y         # SYMBOL AMOUNT pairs (parallel)
rwa --json gm sell-basket SPY 5 TSLA 3 NVDA all  -y
rwa --json gm close-all -y                                   # sell ALL positions (parallel, ~2-8s)
rwa --json gm close-all 50% -y                               # sell 50% of every position
rwa --json gm pnl                                            # avg entry + realized/unrealized P&L
rwa --json gm reclaim                                        # close empty accounts, reclaim rent
```

- `close-all` skips positions < $1.50 (MMs reject tiny swaps) and lists them separately.
- Slippage: default 100 bps; hard-blocked above 3%. Amounts: exact `100`, `50%`, or `all`.
- `search` items carry optional `asset_class`/`region`; `--tag` matches any Ondo tag label (24 factor labels incl. Large Cap, Dividend, High Yield).

## Key JSON

```json
// buy -y / --dry-run (status "success" | "dry_run")
{"status":"success","amount":"0.258","token":"TSLAon","counter_amount":"100.00","counter_token":"USDC","tx":"https://solscan.io/tx/...","slippage_pct":-0.39}
// buy-basket → bought[]+total_usdc_spent · sell-basket → sold[]+total_usdc_received · close-all → sold[]+total_usdc
{"status":"success","sold":[{"token":"TSLAon","amount":"0.25","usdc":"96.50","tx":"..."}],"failed":[],"total_usdc":"96.50"}
```

`status:"success"` only means the run finished — **always check `failed[]`** (always present). Partial success is normal.
An optional `gas_refuel: {"usdc":"5","sol":"0.02...","tx":"..."}` object appears when the CLI auto-bought SOL for fees before the trade (normal for USDC-only wallets — not an error).

## Errors → action

| Error / error_kind | Agent action |
|---|---|
| `market_closed` | Weekend/holiday and this token doesn't trade off-hours. Flagships (TSLAon, NVDAon, SPYon, QQQon, GOOGLon…) trade 24/7 — check `gm hours --tradable` |
| `not_tradable` | Skip token; verify with `gm tradable <SYM>` |
| `slippage_too_high` | Reduce size or skip; MM cooldown after rapid buy+sell — wait 30–60s |
| `amount_below_minimum` | Use ≥ 5 USDC per token |
| `insufficient_funds` (SOL) | Non-gasless route needs ~0.002 SOL. The CLI normally auto-refuels from USDC; if it couldn't, retry (gasless route may fill) or fund SOL |
| `insufficient_funds` (USDC) | Tell user to fund USDC |
| `no_position` / `Balance is 0` | No position to sell |
| `route_unfillable` | Every market maker declined this size right now — try a larger amount or wait |
| Swap failed code -1000/-2003/-2004/-2005 | CLI already auto-retried — do NOT retry manually |
| RPC `unavailable` (exit 75) | Transient — wait a few seconds; on repeats set `RWA_RPC_URL` to a dedicated endpoint |

Exit code **75** = transient, safe to retry the command; **1** = permanent, don't.

## Sessions & liquidity

Sessions (ET): Pre-Market 4:00, Regular 9:30, Post-Market 16:00, Overnight 20:00. **Weekends/holidays are off-hours: only flagship tokens trade** (`gm hours --tradable` shows the live set). Outside Regular hours many small caps are illiquid — on repeated `route_unfillable`/`-2004`, substitute a token or wait; don't hammer the same one.

## Safe pre-trade flow

```bash
rwa keys show && rwa --json gm portfolio && rwa --json gm hours   # wallet, funds, market open?
rwa --json gm buy TSLA 100 -y
```
