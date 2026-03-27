---
name: rwa-portfolio
description: >
  View portfolio holdings, P&L, allocation, and price history for tokenized stocks on Solana.
  Triggers: "show portfolio", "my holdings", "positions", "how much TSLA",
  "price history", "P&L", "24h change", "balance".
---

# Portfolio

```bash
rwa --json gm portfolio                  # Own wallet
rwa --json gm portfolio <WALLET_ADDR>    # Any public wallet
```

Returns: `wallet`, `sol`, `usdc`, `positions[]` (token, balance, price, value_usd, alloc_pct, change_pct_24h), `total_value_usd`, `change_24h_usd`, `change_24h_pct`.

# Price history

```bash
rwa --json gm history TSLA               # Default: 1 month
rwa --json gm history TSLA -r 1D         # Ranges: 1D, 1W, 1M, 3M, 1Y, ALL
```

Returns: `symbol`, `range`, `candles[]`, `first`/`last` (timestamp + price), `high`, `low`, `change_pct`.

# Efficient patterns

- `portfolio` gives the full picture — no need to call `list` or `hours` first
- `history -r 1D` (~50 tok) is much cheaper than `ALL` (~200 tok)
- To check one position: read from `portfolio` instead of `list --search` + `portfolio`
