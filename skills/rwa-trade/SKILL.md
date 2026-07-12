---
name: rwa-trade
description: >
  Buy and sell tokenized stocks & ETFs on Solana via the rwa CLI.
  Use when the user wants to trade, buy, sell, preview a quote, check market
  hours, or list tokens. Triggers: "buy TSLA", "sell stocks", "trade RWA",
  "quote NVDA", "market hours", "list tokens", "what can I buy on Solana".
---

# RWA Trade

Buy/sell 438 tokenized stocks & ETFs (Ondo Global Markets) on Solana. Always pass `--json`; add `-y` to execute, `--dry-run` to preview. **Since v0.6.0 `--json` без `-y` НЕ исполняет** buy/sell/send/baskets/close-all (fails closed). An otherwise-valid trade → `error_kind: confirmation_required`; a precondition failure (`insufficient_funds`/`no_position`/`amount_below_minimum`) surfaces its own kind first, and `close-all` on an empty wallet → `status:success`. Guarantee is "never executes", not "always confirmation_required". `reclaim` runs without confirmation (rent back to your own wallet).

## Golden rules

- **NEVER use `&`, background, or parallel shell processes** — one `rwa` process at a time (a lock enforces it). Multi-token commands parallelize internally.
- Minimum buy: **5 USDC** (single buy and per basket item).
- No `quote` command — preview with `--dry-run`.
- `--quote-only` (buy) quotes any size without a balance check; never executes (rejects `-y`). JSON `status` is `dry_run`. Use to size a buy before funding.
- `sell` swaps tokens → USDC; `send` transfers assets out. Never confuse them.
- CLI auto-retries transient swap failures. **Do NOT retry manually** unless it reports failure after retries.
- Multi-token = `buy-basket` / `sell-basket` / `close-all` — they run **in parallel by default** (bounded internally). Never loop `buy`/`sell` by hand. `--sequential` is a rate-limit fallback only. Launches are staggered + adaptive (real runs and dry-run alike) to dodge Jupiter's per-wallet 429 — live: 5-token basket ≈ 3 s. Setting `RWA_JUPITER_API_KEY` auto-selects the faster keyed profile; no manual tuning.
- Bulk filtering = `gm search` flags. **Never** pipe `gm list` through ad-hoc Python.

## Commands

```bash
rwa --json gm hours                                          # session + tradable_count + paused_count + offhours flagships; next_session_at = epoch seconds (schedule on it)
rwa --json gm search --tradable-only --sector Technology     # bulk scan
rwa --json gm search --tag asia --tag "fixed income"         # any Ondo tag: region, asset class, factor
rwa --json gm tradable TSLA NVDA                             # check specific symbols
rwa --json gm buy  TSLA 100 --dry-run                        # preview (100 USDC in)
rwa --json gm buy  TSLA 100 -y --slippage 50                 # execute, max 0.5% slippage
rwa --json gm sell TSLA all  -y                              # sell all (also 50% / exact)
rwa --json gm buy  TSLA 100 --limit-price 400 -y             # conditional: fills only if quote <= 400 USDC/token
rwa --json gm sell TSLA all --limit-price 450 -y             # conditional: fills only if quote >= 450
rwa --json gm buy  SPY 100 --limit-price 748 share -y        # limit per underlying SHARE (bare number = per token)
rwa --json gm buy  TSLA 100 --max-bps 30 -y                  # reject if all-in cost (spread+fee) > 30 bps
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

## Conditional orders (synthetic limit orders)

`--limit-price <P>` (buy/sell only) executes **only if the quoted price** (USDC per token) is ≤ P for buy / ≥ P for sell (equality fills); otherwise the command exits 1 with `error_kind: condition_not_met` — same in `--dry-run`. The quote that passes the check is the one executed, so worst-case fill = limit ± slippage; pair with a tight `--slippage 20`. JSON echoes `limit_price` on success. Conflicts with `--quote-only`.

Unit: bare number = per **token**; add `share` (`--limit-price 748 share`; joined `748share` also accepted) to gate per underlying **share** instead. GM tokens are total-return trackers (dividends reinvested), so token price = share price × multiplier — previews expose `share_price`/`shares_per_token` once the multiplier drifts from 1. If the multiplier can't be read for a `share`-limit, the command fails closed with exit 75 (transient — retry next tick).

- **Limit order:** schedule `rwa --json gm buy TSLA 100 --limit-price 400 --slippage 20 -y` (cron/loop, every N min) until it exits 0. `condition_not_met` = keep waiting, NOT an error to escalate.
- **DCA:** plain scheduled `buy <SYM> <AMT> -y` — no limit-price needed.
- **Stop-loss:** check the reference price yourself, then market `sell -y`. Do NOT use `--limit-price` for stops (a stop wants out, not a price guarantee).

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
| `condition_not_met` | `--limit-price` unmet — keep the order scheduled, retry next tick; not a failure |
| `confirmation_required` | `--json` without `-y` — add `-y` to execute or `--dry-run` to preview; never retry as-is |
| `trading_paused` | Ondo paused this asset: a dividend window OR the market is closed for it (weekends flag most non-24/7 tokens, so a weekend non-flagship gives `trading_paused`, not `market_closed`). Check `gm hours`/`next_session_at` for when it resumes — don't hammer-retry a weekend pause. In baskets → `failed[]`; close-all skips it |
| `cost_too_high` | Quoted all-in cost exceeds `--max-bps` — raise the ceiling or wait for a tighter spread |
| `amount_below_minimum` | Use ≥ 5 USDC per token |
| `insufficient_funds` (SOL) | Non-gasless route needs ~0.002 SOL. The CLI normally auto-refuels from USDC; if it couldn't, retry (gasless route may fill) or fund SOL |
| `insufficient_funds` (USDC) | Tell user to fund USDC |
| `no_position` / `Balance is 0` | No position to sell |
| `unknown_token` | Symbol not in the GM list (typo?) — find it with `gm search --search <keyword>`; don't retry as-is |
| `invalid_amount` / `invalid_address` | Bad user input (amount is a number, `NN%`, or `all`; address must be valid Solana) — fix, don't retry |
| `lock_contention` (exit 75) | Another rwa process holds the lock — the lock covers EVERY invocation (even reads), so serialize ALL rwa calls; wait and retry |
| `route_unfillable` | No fillable route after retries — now rare, since RFQ makers fund fills just-in-time. Try a larger amount or wait. (A stderr `note: RFQ maker funds just-in-time…` during a buy is NORMAL, not an error — the swap still lands.) |
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
