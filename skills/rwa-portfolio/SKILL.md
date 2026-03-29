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

## JSON shape (v0.2.0+)

```json
{
  "wallet": "5CjgV1J2...",
  "cash": {
    "sol": 0.0870,
    "usdc": 0.00
  },
  "gm_positions": {
    "positions": [
      {
        "token": "TSLAon",
        "balance": 0.2500,
        "price": 385.75,
        "value_usd": 96.44,
        "gm_alloc_pct": 100.00,
        "change_pct_24h": 1.23
      }
    ],
    "value_usd": 96.44,
    "change_24h_usd": 1.17,
    "change_24h_pct": 1.23
  }
}
```

`gm_alloc_pct` = allocation within GM positions only (not total portfolio including cash).

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
- To check one position: read from `gm_positions.positions[]` — no separate balance call needed
- To decide what to sell: sort by `change_pct_24h` or `value_usd` from `portfolio` result
- SOL balance is at `cash.sol`, USDC at `cash.usdc`
