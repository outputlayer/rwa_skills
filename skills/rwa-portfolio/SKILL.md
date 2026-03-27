---
name: rwa-portfolio
description: >
  View portfolio holdings, P&L, allocation, and price history for tokenized stocks on Solana.
  Triggers: "show portfolio", "my holdings", "positions", "how much TSLA",
  "price history", "token performance", "P&L", "24h change", "balance".
---

# Portfolio

```bash
rwa --json gm portfolio                  # Own wallet
rwa --json gm portfolio <WALLET_ADDR>    # Any public wallet
```

Returns: `wallet`, `sol`, `usdc`, `positions[]` (token, balance, price, value_usd, alloc_pct, change_pct_24h), `total_value_usd`, `change_24h_usd`, `change_24h_pct`. Floats rounded: USD to 2dp, balances to 4dp.

# Price history

```bash
rwa --json gm history TSLA               # Default: 1 month
rwa --json gm history TSLA -r 1D         # Ranges: 1D, 1W, 1M, 3M, 1Y, ALL
```

Returns: `symbol`, `range`, `candles`, `first`/`last` (timestamp + price), `high`, `low`, `change_pct`.

# Token-efficient patterns

- `portfolio` alone gives full picture (~41–300 tokens) — no need to call `list` or `hours` first
- `history -r 1D` returns ~50 tokens — much cheaper than `ALL` (~200 tokens with many candles)
- To check a single position: look at `portfolio` result rather than calling `list --search` + `portfolio` separately
