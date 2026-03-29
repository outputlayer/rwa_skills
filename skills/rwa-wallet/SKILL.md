---
name: rwa-wallet
description: >
  Set up Solana wallets and transfer funds for trading tokenized stocks via rwa CLI.
  Triggers: "create wallet", "generate keys", "import seed phrase", "show address",
  "install rwa", "send SOL", "send USDC", "withdraw", "transfer", "reclaim rent",
  "encrypt wallet", "passphrase".
---

# Install

```bash
curl -fsSL https://raw.githubusercontent.com/outputlayer/rwa_cli/main/install.sh | bash
```

# Best defaults

- Prefer encrypted wallets for new setups
- Use `rwa keys show` before asking the user to fund the wallet
- Use `rwa --json` for send/reclaim when an agent is driving
- For post-sell withdrawals, prefer exact values from CLI output

# Wallet

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

# Shortest path

- Need wallet address: `rwa keys show`
- New wallet: `rwa keys generate --encrypt`
- Import existing wallet: `rwa keys import ... --encrypt`
- Send everything after liquidation: `send USDC all` then `send SOL all`
- Need rent back after sells: `reclaim`

# Reclaim rent

```bash
rwa --json gm reclaim
rwa --json gm reclaim --token TSLA
```

Run this after selling or transferring out tokens to recover locked SOL from empty accounts.

# Token efficiency

| Need | Cheapest useful call |
|------|----------------------|
| wallet address | `keys show` |
| create wallet | `keys generate --encrypt` |
| preview transfer | `gm send ... --dry-run` |
| drain USDC | `gm send USDC all` |
| drain SOL | `gm send SOL all` |
| reclaim empty accounts | `gm reclaim` |

# Notes

- `send` moves tokens to another wallet; it does not swap to USDC
- `send USDC all` uses full on-chain precision
- `send SOL all` auto-reserves tx fees
- After `close-all`, use `reclaim` before final `send SOL all`
- For faster/private RPC, set `RWA_RPC_URL`

```bash
export RWA_RPC_URL=https://your-rpc-endpoint.com
```
