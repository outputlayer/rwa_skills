# RWA Skills

[Agent Skills](https://agentskills.io/) for trading tokenized stocks & ETFs on Solana via the [rwa CLI](https://github.com/outputlayer/rwa_cli).

## Quick Start

Best setup for an agent:

```bash
curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh | sh
npx skills add outputlayer/rwa_skills -g -y
```

Then tell the agent the task directly:

```
Buy $100 of TSLA
Show my portfolio
Create an encrypted wallet
Sell everything and withdraw
```

## Install

```bash
npx skills add outputlayer/rwa_skills -g -y
```

**Cursor:** Settings → Rules → Add Rule → Remote Rule → `outputlayer/rwa_skills`

**Manual:** Clone and copy skill folders to your agent's skills directory (`~/.claude/skills/`, `~/.cursor/skills/`, etc.)

## Skills

| Skill | Description |
|-------|-------------|
| **rwa-trade** | Buy/sell, conditional orders (`--limit-price`), dry-run previews, market hours, bulk buy/sell, close-all, tradable check |
| **rwa-portfolio** | Holdings, P&L, allocation, price history |
| **rwa-wallet** | Wallet setup, encryption, send/withdraw, reclaim rent |

## Design goals

- Short decision rules for agents
- Prefer `rwa --json` everywhere
- Add a canonical example for every common workflow
- Treat `--dry-run` as the default preview path for state-changing commands
- Favor the cheapest useful command first — use `gm tradable` for symbol checks and `gm search` for bulk scans
- Avoid parallel wallet-changing *processes* (multi-token commands parallelize internally)
- Preserve exact CLI amount precision; never manually round inputs
- Encode bulk-buy and bulk-sell best practices directly in the skills
- Keep portfolio answers honest about `cash.*` vs `gm_positions.*`; never treat GM totals as full wallet totals
- Prefer surfaced error kinds like `market_closed`, `not_tradable`, and `slippage_too_high` over brittle string matching

## Canonical examples

Preview then buy:

```bash
rwa --json gm buy TSLA 100 --dry-run
rwa --json gm buy TSLA 100 -y
```

Preview then sell:

```bash
rwa --json gm sell TSLA 50% --dry-run
rwa --json gm sell TSLA 50% -y
```

Buy multiple tokens — check tradable + buy in 2 commands:

```bash
# 1. Get all tradable tokens in one call, filter by sector
rwa --json gm search --tradable-only --sector Healthcare --type stock

# 2. Buy all at once — per-token amounts, parallel
rwa --json gm buy-basket JNJ 25 LLY 25 PFE 25 ABBV 25 -y   # parallel by default

# ...or split one USDC total by percent weights (must sum to 100; each item ≥ 5 USDC)
rwa --json gm buy-basket TSLA 50% NVDA 30% SPY 20% --total 1000 -y
```

Sell specific positions:

```bash
rwa --json gm sell-basket SPY 5 TSLA 3 NVDA all -y
```

Synthetic limit order — schedule until it fills (exit 0; `condition_not_met` = keep waiting). Bare number gates per token; `--limit-price 748 share` gates per underlying share:

```bash
rwa --json gm buy TSLA 100 --limit-price 400 --slippage 20 -y
```

Portfolio lookup:

```bash
rwa --json gm portfolio
```

Withdraw after liquidation:

```bash
rwa --json gm close-all -y
rwa --json gm reclaim
rwa --json gm send USDC all <ADDR> -y
rwa --json gm send SOL all <ADDR> -y
```

## Links

- [RWA CLI](https://github.com/outputlayer/rwa_cli)
- [Ondo Global Markets](https://ondo.finance/)
- [Agent Skills Spec](https://agentskills.io/specification)

## License

MIT
