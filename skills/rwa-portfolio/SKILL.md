---
name: rwa-portfolio
description: >
  View portfolio holdings, P&L, allocation, and price history for tokenized stocks on Solana.
  Triggers: "show portfolio", "my holdings", "positions", "how much TSLA",
  "price history", "P&L", "24h change", "balance", "what do I own".
---

# Portfolio

```bash
rwa --json gm portfolio                   # Own wallet
rwa --json gm portfolio <WALLET_ADDR>     # Any public wallet
```

Returns: `wallet`, `sol`, `usdc`, `positions[]` (token, balance, price, value_usd, alloc_pct, change_pct_24h), `total_value_usd`, `change_24h_usd`, `change_24h_pct`.

# Price history

```bash
rwa --json gm history TSLA                # Default: 1 month
rwa --json gm history TSLA -r 1D         # Ranges: 1D, 1W, 1M, 3M, 1Y, ALL
```

Returns: `symbol`, `range`, `candles[]`, `first`/`last` (timestamp + price), `high`, `low`, `change_pct`.

# Token cost per response

| Command | ~Tokens |
|---------|---------|
| `portfolio` (no positions) | 41 |
| `portfolio` (5 positions) | ~180 |
| `portfolio` (20 positions) | ~500 |
| `history -r 1D` | ~50 |
| `history -r 1M` | ~120 |
| `history -r ALL` | ~200 |

# Efficient patterns

- `portfolio` gives the full picture — no need to call `list` or `hours` first
- `history -r 1D` (~50 tok) is much cheaper than `ALL` (~200 tok) for recent moves
- To check one position: read `balance` and `value_usd` from `portfolio` instead of separate calls
- To decide what to sell: `portfolio` → inspect `change_pct_24h` and `value_usd` per position
