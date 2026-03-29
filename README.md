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
| **rwa-trade** | Buy/sell, dry-run previews, market hours, bulk buy, close-all |
| **rwa-portfolio** | Holdings, P&L, allocation, price history |
| **rwa-wallet** | Wallet setup, encryption, send/withdraw, reclaim rent |

## Design goals

- Short decision rules for agents
- Prefer `rwa --json` everywhere
- Add a canonical example for every common workflow
- Treat `--dry-run` as the default preview path for state-changing commands
- Favor the cheapest useful command first
- Avoid parallel wallet-changing commands
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
