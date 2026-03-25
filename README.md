# RWA Skills

A collection of [Agent Skills](https://agentskills.io/) for trading tokenized stocks & ETFs on Solana via the [rwa CLI](https://github.com/outputlayer/rwa_cli).

Skills are reusable capabilities for AI coding agents. They provide procedural knowledge that helps agents trade tokenized stocks, check portfolios, discover tokens, and more using Ondo Global Markets on Solana.

## Installing

These skills work with any agent that supports the [Agent Skills](https://agentskills.io/) standard.

### npx skills

Install using the [npx skills](https://skills.sh/) CLI:

```
npx skills add outputlayer/rwa_skills --skill rwa -g
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

Skills are contextual and auto-loaded based on your conversation.

| Skill | Description |
|-------|-------------|
| **rwa** | Trade 264 tokenized stocks & ETFs (Ondo Global Markets) on Solana — buy, sell, quote, portfolio, price history via the rwa CLI |

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
