---
name: rwa-portfolio
description: >
  View portfolio holdings, P&L, allocation, and price history for tokenized stocks on Solana.
  Triggers: "show portfolio", "my holdings", "positions", "how much TSLA",
  "price history", "P&L", "24h change", "balance", "what do I own".
---

# Shortest path

- User asks what they own: `portfolio`
- User asks about one token they hold: still start with `portfolio`
- User asks for chart/history: `history`
- Do not call `list` or `hours` before `portfolio` unless the user asked about market status

# Portfolio

```bash
rwa --json gm portfolio
rwa --json gm portfolio <WALLET_ADDR>
```

Returns:

- `wallet`
- `sol`
- `usdc`
- `positions[]`
- `gm_positions_value_usd`
- `gm_positions_change_24h_usd`
- `gm_positions_change_24h_pct`

Use `positions[].balance`, `value_usd`, `gm_alloc_pct`, and `change_pct_24h` to answer most follow-ups without extra calls.

Important: `sol` and `usdc` are separate cash balances. GM totals use explicit `gm_*` field names, so they are not mistaken for full wallet totals.

# Price history

```bash
rwa --json gm history TSLA
rwa --json gm history TSLA -r 1D
rwa --json gm history TSLA -r 1W
rwa --json gm history TSLA -r 1M
rwa --json gm history TSLA -r 3M
rwa --json gm history TSLA -r 1Y
rwa --json gm history TSLA -r ALL
```

# Token efficiency

| Need | Cheapest useful call |
|------|----------------------|
| full holdings | `portfolio` |
| one position size | `portfolio` |
| recent move | `history -r 1D` |
| default trend | `history` |
| long history | `history -r ALL` |

# Good patterns

- "How much TSLA do I have?" -> `portfolio`, then filter `positions`
- "What should I sell?" -> `portfolio`, inspect `value_usd` and `change_pct_24h`
- "Show my allocation" -> `portfolio` only
- "Show price action today" -> `history -r 1D`
- "What is my full wallet worth?" -> `portfolio`, then explain that cash is separate from GM totals

# Notes

- `portfolio` is the best first call for almost every holdings question
- `history -r 1D` is much cheaper than `ALL`
- Avoid repeated `history` calls for many symbols unless the user explicitly wants them
- Do not describe `gm_alloc_pct` as total wallet allocation when large `USDC`/`SOL` cash balances matter
