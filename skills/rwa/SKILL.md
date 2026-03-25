---
name: rwa
description: >
  Trade 264 tokenized stocks & ETFs (Ondo Global Markets) on Solana via the rwa CLI.
  Use when user asks about tokenized stocks, RWA trading, Ondo GM tokens, buying/selling
  TSLA/AAPL/NVDA/SPY on Solana, checking portfolio of tokenized assets, or says
  "buy stocks on Solana", "trade RWA", "Ondo portfolio".
compatibility: Requires network access and the rwa CLI (auto-installed on first use). Works on macOS and Linux.
allowed-tools: Bash(rwa:*) Bash(curl:*) Read
metadata:
  author: outputlayer
  version: "1.0"
---

# RWA CLI

A command-line interface for trading tokenized stocks & ETFs ([Ondo Global Markets](https://ondo.finance/)) on Solana via Jupiter.

264 tokens available — TSLA, AAPL, NVDA, SPY, QQQ, and more.

## Prerequisites

Assume the rwa CLI is already installed. Do not run upfront install checks. Just execute the requested `rwa` command directly.

If a `rwa` command fails, inspect the error:

- "command not found" → CLI not installed. See [install-and-recovery.md](references/install-and-recovery.md#cli-not-found).
- "Solana RPC unavailable" → Rate limited. Retry in a few seconds, or set `RWA_RPC_URL` to a private endpoint. See [install-and-recovery.md](references/install-and-recovery.md#rpc-errors).
- "wallet not found" → No wallet configured. Run `rwa keys generate` or `rwa keys import`.

## Global Flags

| Flag | Description |
|------|-------------|
| `--json` | Machine-readable JSON output (use on every command for agent workflows) |
| `-y` | Skip confirmation on buy/sell (required for non-interactive agents) |
| `--rpc-url <URL>` | Custom Solana RPC endpoint (or set `RWA_RPC_URL` env var) |

> **Always use `--json` on every command.** JSON output contains full detail and is unambiguous to parse.
> The default text format is for human terminal use.

## Command Overview

| Command | Description | Needs wallet |
|---------|-------------|:---:|
| `rwa gm hours` | Market status (OPEN/CLOSED + countdown) | No |
| `rwa gm list` | All 264 available tokens | No |
| `rwa gm quote <SYM> <AMT>` | Swap quote (buy with USDC) | No |
| `rwa gm quote <SYM> <AMT> --sell` | Swap quote (sell for USDC) | No |
| `rwa gm buy <SYM> <AMT> -y` | Execute buy | Yes |
| `rwa gm sell <SYM> <AMT> -y` | Execute sell | Yes |
| `rwa gm portfolio [WALLET]` | Holdings + allocation + 24h change | No* |
| `rwa gm history <SYM> [-r RANGE]` | Price history (1D/1W/1M/3M/1Y/ALL) | No |
| `rwa keys generate` | Create new Solana wallet | No |
| `rwa keys import --seed-phrase "..."` | Import from mnemonic | No |
| `rwa keys show` | Show address + key file path | Yes |

\* Portfolio works with any public wallet address as argument; local wallet needed only if no address provided.

## Amount Formats

- Exact: `100` (USDC for buy, tokens for sell)
- Percentage: `50%` (half of balance)
- All: `all` (entire balance)

## Common Workflows

### Check Market and Discover Tokens

```bash
# Check if market is open (24/5, Sun 8pm – Fri 8pm ET)
rwa --json gm hours

# List all available tokens
rwa --json gm list
```

### Get a Quote Before Trading

```bash
# How much TSLA do I get for 100 USDC?
rwa --json gm quote TSLA 100

# How much USDC do I get for selling 5 TSLA?
rwa --json gm quote TSLA 5 --sell
```

### Execute a Trade

```bash
# Buy $100 worth of Tesla
rwa gm buy TSLA 100 -y

# Sell all NVDA holdings
rwa gm sell NVDA all -y

# Sell 50% of SPY
rwa gm sell SPY 50% -y
```

### View Portfolio

```bash
# Own wallet
rwa --json gm portfolio

# Any public wallet
rwa --json gm portfolio Dn9EqxugBePrno7gzCjbGeYxY3VJE9RB2WE2FH7t7qmH
```

### Price History

```bash
rwa --json gm history TSLA -r 1M
rwa --json gm history SPY -r 1Y
```

## Key JSON Outputs

```json
// rwa --json gm hours
{"status":"OPEN","next_close":"2026-03-28T00:00:00Z","countdown":"2d 15h 30m"}

// rwa --json gm portfolio
{"wallet":"...","sol":1.5,"usdc":500.0,"positions":[{"token":"TSLAon","balance":0.26,"price":385.0,"value_usd":100.1,"alloc_pct":15.2,"change_pct_24h":1.2}],"total_value_usd":600.1}

// rwa --json gm list
[{"symbol":"TSLAon","name":"Tesla","mint":"..."},...]
```

## Token Notes

- Both `TSLA` and `TSLAon` symbol formats accepted
- Ondo GM tokens are **total return trackers** — dividends reinvested, 1 token ≠ 1 share
- All tokens use Solana Token-2022 program
- Fund wallet with SOL (gas ~0.01 SOL per tx) + USDC (trading) before first trade

## Security

- Never output wallet private keys or seed phrases in responses
- Always use `-y` flag for non-interactive execution, but confirm trade details with user first for large amounts
- Always use `--json` for reliable parsing
- If RPC errors occur, retry or suggest `RWA_RPC_URL` for a private endpoint

## Reference Documents

Load the relevant reference when you need detailed information:

| When you need... | Load |
|---|---|
| CLI install, wallet setup, RPC troubleshooting | [install-and-recovery.md](references/install-and-recovery.md) |
| Ondo GM token details, trading mechanics | [ondo-gm-tokens.md](references/ondo-gm-tokens.md) |
