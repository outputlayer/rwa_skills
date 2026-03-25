---
name: rwa-trade
description: >
  Buy and sell tokenized stocks & ETFs on Solana via the rwa CLI.
  Use when user wants to trade, buy, sell, get a quote, check market hours,
  or list available tokens. Triggers: "buy TSLA", "sell stocks", "trade RWA",
  "quote NVDA", "market hours", "list tokens", "what can I buy on Solana".
compatibility: Requires network access and the rwa CLI (auto-installed on first use). Works on macOS and Linux.
allowed-tools: Bash(rwa:*) Bash(curl:*) Read
metadata:
  author: outputlayer
  version: "1.0"
---

# RWA Trade

Buy and sell 264 tokenized stocks & ETFs ([Ondo Global Markets](https://ondo.finance/)) on Solana.

## Prerequisites

Assume `rwa` is installed. If "command not found", install:
```bash
curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh | bash
```

## Flags

| Flag | Required for agents |
|------|:---:|
| `--json` | Yes — always use for machine-readable output |
| `-y` | Yes — skip confirmation on buy/sell |
| `--rpc-url <URL>` | No — custom Solana RPC (or `RWA_RPC_URL` env) |

## Workflow

### 1. Check Market Hours

Trading is 24/5: Sunday 8 PM — Friday 8 PM ET.

```bash
rwa --json gm hours
```

### 2. Find Tokens

```bash
rwa --json gm list       # All 264 tokens
```

Both `TSLA` and `TSLAon` symbol formats accepted.

### 3. Get a Quote

```bash
rwa --json gm quote TSLA 100          # Buy: 100 USDC → TSLA
rwa --json gm quote TSLA 5 --sell     # Sell: 5 TSLA → USDC
```

### 4. Execute Trade

```bash
rwa gm buy TSLA 100 -y       # Buy with USDC
rwa gm sell TSLA all -y       # Sell entire position
rwa gm sell SPY 50% -y        # Sell half
```

## Amount Formats

- Exact: `100` (USDC for buy, tokens for sell)
- Percentage: `50%`
- All: `all`

## Key JSON Output

```json
// rwa --json gm quote TSLA 100
{"input":"USDC","output":"TSLAon","in_amount":100.0,"out_amount":0.26,"price":385.0}
```

## Errors

- "market closed" → check `rwa --json gm hours`, wait for open
- "Solana RPC unavailable" → retry in a few seconds, or set `RWA_RPC_URL`
- "No wallet found" → run `rwa keys generate` first
- "Insufficient SOL for gas" → send ≥0.005 SOL to wallet
- "Insufficient USDC" → send USDC to wallet before buying
- "Balance is 0 — nothing to trade" → wallet has no tokens to sell
- "Minimum buy amount is 1.0 USDC" → amount too small

## Safety — Pre-Trade Checklist

The CLI enforces these checks automatically, but agents should verify upfront to give clear guidance:

1. **Wallet exists**: `rwa keys show` — if fails, run `rwa keys generate` first
2. **Has SOL for gas**: portfolio must show `sol >= 0.005`
3. **Has USDC for buys**: portfolio must show `usdc >= buy amount`
4. **Market is open**: `rwa --json gm hours` must return `"status":"OPEN"`

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
```

If any step fails, tell the user what's missing before attempting the trade.
