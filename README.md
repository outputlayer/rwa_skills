# RWA Skills

A collection of [Agent Skills](https://agentskills.io/) for trading tokenized stocks & ETFs on Solana via the [rwa CLI](https://github.com/outputlayer/rwa_cli).

Skills are reusable capabilities for AI coding agents. They provide procedural knowledge that helps agents trade tokenized stocks, check portfolios, manage wallets, and more using Ondo Global Markets on Solana.

## Quick Start — copy this to any AI agent

```
I'd like to trade tokenized stocks on Solana.

Install skills if npm is available: npx skills add outputlayer/rwa_skills -g

Otherwise install the CLI directly: curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh | bash
```

The agent will install the tools, create a wallet, and guide you through your first trade.

## Installing

These skills work with any agent that supports the [Agent Skills](https://agentskills.io/) standard.

### npx skills

Install using the [npx skills](https://skills.sh/) CLI:

```
npx skills add outputlayer/rwa_skills -g
```

### Cursor

Add via Settings → Rules → Add Rule → Remote Rule (GitHub) with `outputlayer/rwa_skills`.

### Manual install

Clone this repo and copy the skill folders into the appropriate directory for your agent:

| Agent | Directory | Docs |
|-------|-----------|------|
| Claude Code | `~/.claude/skills/` | [docs](https://docs.anthropic.com/en/docs/claude-code) |
| Cursor | `~/.cursor/skills/` | [docs](https://docs.cursor.com) |
| OpenCode | `~/.config/opencode/skills/` | [docs](https://opencode.ai) |
| OpenAI Codex | `~/.codex/skills/` | [docs](https://openai.com/codex) |

## Skills

Skills are contextual and auto-loaded based on your conversation. Only the relevant skill is loaded — keeping context lean.

| Skill | Description | Triggers |
|-------|-------------|----------|
| **rwa-trade** | Buy/sell tokenized stocks, get quotes, check market hours, list tokens | "buy TSLA", "sell stocks", "quote", "market hours" |
| **rwa-portfolio** | View holdings, P&L, allocation, price history | "portfolio", "holdings", "price history" |
| **rwa-wallet** | Create/import wallets, install CLI, configure RPC | "create wallet", "import keys", "install rwa" |

## Skill Structure

Each skill follows the [Agent Skills specification](https://agentskills.io/specification):

```
skill-name/
├── SKILL.md           # Required. Instructions for the agent (with YAML frontmatter)
├── references/        # Optional. Supporting documentation loaded on demand
└── assets/            # Optional. Static resources
```

- `SKILL.md` — The entry point. Contains YAML frontmatter and markdown instructions.
- `references/` — Detailed docs loaded only when needed, keeping context usage efficient.

## Resources

- [RWA CLI](https://github.com/outputlayer/rwa_cli) — the CLI tool
- [Ondo Finance](https://ondo.finance/) — Ondo Global Markets
- [Jupiter](https://jup.ag/) — Solana DEX aggregator
- [Agent Skills Specification](https://agentskills.io/specification)

## License

MIT — see [LICENSE](LICENSE) for details.
