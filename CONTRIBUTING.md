# Contributing to RWA Skills

## Adding a New Skill

1. Create `skills/<skill-name>/SKILL.md`
2. Add YAML frontmatter with `name` and `description` (with trigger phrases)
3. Keep it under 80 lines — every token counts in agent context
4. Update the table in `README.md`

### SKILL.md Structure

```markdown
---
name: rwa-example
description: >
  One-line purpose.
  Triggers: "keyword1", "keyword2", "keyword3".
---

# Section

Rules, commands, examples — concise, no filler.
```

### Rules for Skill Content

- **Concise over complete** — agents have limited context windows. Aim for <2000 tokens.
- **Commands first** — lead with `bash` code blocks, not explanations.
- **Always use `--json -y`** — skills target agents, not humans.
- **Include error table** — map common errors to agent actions (skip, retry, tell user).
- **Add token cost hints** — helps agents pick efficient command sequences.
- **No duplicate info** — if `rwa-trade` covers buy/sell, don't repeat it in `rwa-portfolio`.

### Testing Your Skill

1. Copy to your agent's skills directory:

   ```bash
   cp -r skills/rwa-example ~/.agents/skills/
   ```

2. Ask your agent to perform a task that matches the trigger phrases
3. Verify the agent uses the correct commands with `--json -y`
4. Check edge cases: market closed, invalid symbol, insufficient balance

## Modifying Existing Skills

- Don't break the YAML frontmatter — agents won't load the skill
- Keep description triggers accurate — agents match on these
- Test with multiple agents (Claude, Cursor, Copilot) if possible
- Run markdown lint before committing: `npx markdownlint-cli2 "**/*.md"`

## Pull Request Checklist

- [ ] SKILL.md has valid YAML frontmatter (`name` + `description` with triggers)
- [ ] Skill is under 80 lines
- [ ] Commands use `--json -y` flags
- [ ] Error table included for commands that can fail
- [ ] README.md table updated
- [ ] `npx markdownlint-cli2 "**/*.md"` passes

## Common Edge Cases to Document

| Scenario | Expected Agent Behavior |
|----------|------------------------|
| Market closed | Show next open time, don't retry |
| Token not tradable | Skip, suggest `hours --tradable` |
| Insufficient balance | Show current balance, don't retry |
| RPC errors | Wait 5s, retry up to 3x, then suggest `RWA_RPC_URL` |
| Invalid symbol | Suggest `list --search` |
| Concurrent commands | Never — always sequential with `sleep 3` |
