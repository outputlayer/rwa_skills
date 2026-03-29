---
name: rwa-wallet
description: >
  Set up Solana wallets and transfer funds for trading tokenized stocks via rwa CLI.
  Triggers: "create wallet", "generate keys", "import seed phrase", "show address",
  "install rwa", "send SOL", "send USDC", "withdraw", "transfer", "reclaim rent",
  "encrypt wallet", "passphrase".
---

# Hard rules

- Prefer encrypted wallets for new setups
- Use `rwa --json` for agent-driven send/reclaim flows
- `send` transfers assets; it does not sell them
- After liquidation, run `reclaim` before final `send SOL all`
- For post-sell withdrawals, prefer exact values from CLI output when available

# JSON / dry-run defaults

- Default to `rwa --json` for send and reclaim flows
- Use `--dry-run` for `gm send` when previewing a transfer for the user
- There is no `--dry-run` for `keys` commands; use `keys show` for low-risk inspection
- Do not use `-y` with `--dry-run`

# Intent -> command

| User intent | Preferred command |
|-------------|-------------------|
| install CLI | `curl -fsSL <https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh> \| sh` |
| create new wallet | `rwa keys generate --encrypt` |
| import existing wallet | `rwa keys import ... --encrypt` |
| show wallet address | `rwa keys show` |
| encrypt existing wallet | `rwa keys encrypt` |
| decrypt wallet | `rwa keys decrypt` |
| preview transfer | `rwa --json gm send ... --dry-run` |
| send all USDC | `rwa --json gm send USDC all <ADDR> -y` |
| send all SOL | `rwa --json gm send SOL all <ADDR> -y` |
| reclaim empty accounts | `rwa --json gm reclaim` |

# Do

- Use `keys show` before asking the user to fund the wallet
- Default to encrypted wallet creation/import
- Use `--dry-run` when previewing transfers for the user
- Use `send USDC all` and `send SOL all` for “withdraw everything”

# Don't

- Do not tell the agent to “sell” when the real intent is “send”
- Do not use manual arithmetic for final withdrawal amounts if CLI already returned them
- Do not skip `reclaim` after `close-all` when the goal is full withdrawal

# Install

```bash
curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh | sh
```

# Wallet commands

```bash
rwa keys generate
rwa keys generate --encrypt
rwa keys import --seed-phrase "word1 ... word12"
rwa keys import --seed-phrase "word1 ... word12" --encrypt
rwa keys import --private-key <BASE58>
rwa keys import --file <PATH>
rwa keys show
rwa keys encrypt
rwa keys decrypt
```

# Passphrase

If `~/.config/rwa/key.age` exists, commands prompt for the passphrase.

```bash
export RWA_PASSPHRASE="my passphrase"
rwa --json gm portfolio
```

# Send / withdraw

```bash
rwa --json gm send SOL 1.5 <ADDR> --dry-run
rwa --json gm send SOL all <ADDR> -y
rwa --json gm send USDC 100 <ADDR> --dry-run
rwa --json gm send USDC all <ADDR> -y
rwa --json gm send TSLA 0.5 <ADDR> -y
```

# Canonical examples

Create an encrypted wallet, then show the address:

```bash
rwa keys generate --encrypt
rwa keys show
```

Preview then send USDC:

```bash
rwa --json gm send USDC 100 <ADDR> --dry-run
rwa --json gm send USDC 100 <ADDR> -y
```

Withdraw everything after liquidation:

```bash
rwa --json gm reclaim
rwa --json gm send USDC all <ADDR> -y
rwa --json gm send SOL all <ADDR> -y
```

# Shortest path

- Need wallet address -> `rwa keys show`
- New wallet -> `rwa keys generate --encrypt`
- Import wallet -> `rwa keys import ... --encrypt`
- Withdraw everything after liquidation -> `send USDC all`, then `send SOL all`
- Recover locked SOL -> `reclaim`
- Pure transfer flow -> go straight to `gm send`; do not call `portfolio` unless the user asked about holdings

# Reclaim rent

```bash
rwa --json gm reclaim
rwa --json gm reclaim --token TSLA
```

Run this after selling or transferring out tokens to recover locked SOL from empty accounts.

# Cheapest useful call

| Need | Preferred call |
|------|----------------|
| wallet address | `keys show` |
| create wallet | `keys generate --encrypt` |
| preview transfer | `gm send ... --dry-run` |
| drain USDC | `gm send USDC all` |
| drain SOL | `gm send SOL all` |
| reclaim empty accounts | `gm reclaim` |

# Canonical withdrawal flow

```bash
rwa --json gm close-all -y
rwa --json gm reclaim
rwa --json gm send USDC all <ADDR> -y
rwa --json gm send SOL all <ADDR> -y
```

# Notes

- `send USDC all` uses full on-chain precision
- `send SOL all` auto-reserves tx fees and now uses exact lamport arithmetic
- The install script prefers a release binary and falls back to source install
- For faster/private RPC, set `RWA_RPC_URL`

```bash
export RWA_RPC_URL=https://your-rpc-endpoint.com
```
