---
name: rwa-portfolio
description: >
  View portfolio holdings, P&L, allocation, and price history for tokenized stocks on Solana.
  Triggers: "show portfolio", "my holdings", "positions", "how much TSLA",
  "price history", "P&L", "profit", "average entry", "24h change", "balance", "what do I own".
---

# Portfolio

```bash
rwa --json gm portfolio                   # Own wallet
rwa --json gm portfolio <WALLET_ADDR>     # Any public wallet
```

## JSON shape

```json
{
  "wallet": "5CjgV1J2...",
  "cash": { "sol": 0.0870, "usdc": 0.00 },
  "gm_positions": {
    "positions": [
      { "token": "TSLAon", "balance": 0.2500, "price": 385.75,
        "value_usd": 96.44, "gm_alloc_pct": 100.00, "change_pct_24h": 1.23 }
    ],
    "value_usd": 96.44, "change_24h_usd": 1.17, "change_24h_pct": 1.23
  }
}
```

`gm_alloc_pct` = allocation within GM positions only (not total portfolio including cash).

# P&L (entry prices, realized/unrealized)

```bash
rwa --json gm pnl        # own wallet only (built from the local trade ledger)
```

```json
{
  "wallet": "5CjgV1J2...", "trades_recorded": 12,
  "tokens": [
    { "token": "TSLAon", "qty": "0.25", "avg_cost": 380.10, "market_price": 385.75,
      "invested_usdc": 95.03, "market_value_usdc": 96.44,
      "unrealized_usdc": 1.41, "realized_usdc": 3.20 }
  ],
  "totals": { "invested_usdc": 95.03, "market_value_usdc": 96.44,
              "unrealized_usdc": 1.41, "realized_usdc": 3.20, "total_pnl_usdc": 4.61 }
}
```

Semantics agents must respect:

- Built **only from trades this CLI executed** (every buy/sell is logged locally per wallet). Deposits/withdrawals are cash movements and are ignored by design.
- `avg_cost` = average entry price of the open position; `unrealized_usdc` = market value − invested; `realized_usdc` = locked-in P&L from sells; `total_pnl_usdc` = both.
- A token with an `oversold_qty` field was partly sold beyond CLI-recorded buys (acquired elsewhere) — that part is excluded from P&L, not mispriced.
- Use `portfolio` for what the wallet holds NOW (on-chain truth); use `pnl` for entry prices and profit.
- `pnl` JSON carries `ledger_integrity`: `ok` | `legacy` (pre-chain entries) | `broken@line N` (history may be modified — flag to the user; P&L still computes).
- Balances/prices are in **raw tokens** (the CLI's canonical frame). Dividend-accruing tokens carry an optional `shares_per_token` (e.g. 1.0077): wallets like Phantom display raw × that multiplier, so the CLI's balance can read slightly lower than the wallet's — same value, different unit. Token price = share price × multiplier (total-return).

# Price history

```bash
rwa --json gm history TSLA                # Default: 1 month
rwa --json gm history TSLA -r 1D          # Ranges: 1D, 1W, 1M, 3M, 1Y, ALL
```

Returns: `symbol`, `range`, `candles` count, `first`/`last` (timestamp + price), `high`, `low`, `change_pct`.

# Efficient patterns

- `portfolio` gives the full picture — no need to call `list` or `hours` first
- `history -r 1D` (~50 tok) is much cheaper than `ALL` (~200 tok) for recent moves
- To check one position: read `gm_positions.positions[]` — no separate balance call
- To decide what to sell: sort by `change_pct_24h`/`value_usd` from `portfolio`, or by `unrealized_usdc` from `pnl`
- SOL balance is at `cash.sol`, USDC at `cash.usdc`
