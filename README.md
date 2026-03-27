# RWA Skills

[Agent Skills](https://agentskills.io/) for trading tokenized stocks & ETFs on Solana via the [rwa CLI](https://github.com/outputlayer/rwa_cli).

## Quick Start

Copy this to any AI agent:

```
I'd like to trade tokenized stocks on Solana.

First install the CLI: curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh | bash

Then if npm is available, install skills: npx skills add outputlayer/rwa_skills -g -y
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
| **rwa-trade** | Buy/sell stocks, quotes, market hours, close-all |
| **rwa-portfolio** | Holdings, P&L, allocation, price history |
| **rwa-wallet** | Wallet setup, send/withdraw, reclaim rent |

## Links

- [RWA CLI](https://github.com/outputlayer/rwa_cli)
- [Ondo Global Markets](https://ondo.finance/)
- [Agent Skills Spec](https://agentskills.io/specification)

## License

MIT
